```js
import React, { PureComponent } from "react";
import { connect } from "dva";
import moment from "moment";
import { Card, Row, Col, Spin, Divider, Icon, Table, Input, Button, Select } from "antd";
import PageHeaderLayout from "../../../layouts/PageHeaderLayout";
import DescriptionList from "../../../components/DescriptionList/index";
import TableButton from "../../../components/TableButton";
import DeviceLogModal from "./deviceLogModal";
import { updatePrinterToolInfo, getHardPlugInfo, updateHardPlugInfo } from "../printManageService";
import styles from "./printDetail.less";

const { Description } = DescriptionList;
const { Option } = Select;
let lanIp = "";
let wwanIp = "";

const items = (ctx) => [
  {
    id: "scan_king_sn",
    name: "设备号",
    value: ctx.state.inputValue.scan_king_sn,
  },
  {
    id: "cashier_system_name",
    name: "收银系统名称",
    value: ctx.state.inputValue.cashier_system_name,
  },
  {
    id: "bill_receipt_name",
    name: "票据名称",
    value: ctx.state.inputValue.bill_receipt_name,
  },
  {
    id: "cashier_turnover_name",
    name: "收银系统流水号名称",
    value: ctx.state.inputValue.cashier_turnover_name,
  },
  {
    id: "pay_way",
    name: "支付方式",
    value: ctx.state.inputValue.pay_way,
  },
  {
    id: "amount_position",
    name: "获取金额位置",
    value: ctx.state.inputValue.amount_position,
  },
  {
    id: "receipt_picture_url",
    name: "小票照片上传",
    value: ctx.state.inputValue.receipt_picture_url,
  },
  // {
  //   id: "billReceiptName",
  //   name: "收银系统名称",
  // },
  // {
  //   id: "cashierTurnoverName",
  //   name: "收银系统名称",
  // },
];
const BillType = [
  { name: "先付款后出票", id: "0" },
  { name: "先出票后付款", id: "1" },
];
const InterceptType = [
  { name: "是", id: "1" },
  { name: "否", id: "0" },
];

const options = (ctx) => [
  {
    // label: "学校类别",
    // type: "select",
    id: "bill_type",
    need: "先付款后出票",
    name: "出票模式",
    optionLists: BillType,
    key: "bill_type",
    // onSelectChange: (e) => ctx.handleAssociation(e),
    // isRequired: true,
  },
  {
    // label: "学校类别",
    // type: "select",
    id: "intercept_type",
    name: "仿真打印机",
    optionLists: InterceptType,
    key: "intercept_type",
    // onSelectChange: (e) => ctx.handleAssociation(e),
    // isRequired: true,
  },
];
@connect(({ printDetail, loading }) => ({
  storeInfo: printDetail.storeInfo,
  detailInfo: printDetail.detailInfo,
  statusData: printDetail.statusData,
  deviceLogData: printDetail.deviceLogData,
  relevanceList: printDetail.relevanceList,
}))
export default class PrintDetail extends PureComponent {
  state = {
    isLoading: false,
    deviceLogVisiable: false,
    relevanceListLoading: true,
    inputValue: {
      scan_king_sn: "",
      cashier_system_name: "",
      bill_receipt_name: "",
      cashier_turnover_name: "",
      pay_way: "",
      amount_position: "",
      receipt_picture_url: "",
      bill_first: "",
      intercept_always: "",
    },
    isfilled: false,
  };

  componentDidMount() {
    const { match } = this.props;
    this.setState({ isLoading: true }, () => {
      this.getDeviceStoreInfo();
      this.getDeviceDetailInfo();
      this.getDeviceStatus();
      if (match.params.sn.startsWith("CA25") && match.params.sn.length === 18) {
        this.getRelevanceList();
      }
    });
    this.getDeviceInfo();
    // this.download("https://images.wosaimg.com/1f/83c7440fdee1998937dca7be44e49d09f87445.jpeg");
  }

  columns = [
    { title: "设备号", dataIndex: "relevanceSn" },
    {
      title: "打印机类型",
      dataIndex: "printerType",
      render: (type) => <span>{type === "1" ? "USB打印机" : "网络打印机"}</span>,
    },
    {
      title: "IP地址",
      dataIndex: "localIP",
      render: (text) => <span>{text ? text.split(":")[0] : "--"}</span>,
    },
    { title: "品牌", dataIndex: "deviceProvider" },
    {
      title: "纸宽",
      dataIndex: "paperSpec",
      render: (text) => <span>{text + "mm"}</span>,
    },
    {
      title: "解绑打印机",
      dataIndex: "remove",
      render: (text, record) => (
        <TableButton type="primary" onClick={() => this.removeRelevance(record.relevanceSn)}>
          删除
        </TableButton>
      ),
    },
  ];

  download = (url) => {
    // const fileUrl = url;
    const form = document.createElement("form");
    form.method = "get";
    form.action = url;
    document.body.append(form);
    form.submit();
  };

  getDeviceInfo = async () => {
    const { sn } = this.props.match.params;
    try {
      const { code, data } = await getHardPlugInfo({
        printer_assistant_sn: sn,
      });
      if (code === 200) {
        const {
          scanKingSn,
          cashierSystemName,
          billReceiptName,
          cashierTurnoverName,
          payWay,
          amountPosition,
          receiptUrl,
          billFirst,
          interceptAlways,
        } = data;
        this.setState((prevState) => {
          let inputValue = Object.assign({}, prevState.inputValue);
          inputValue["scan_king_sn"] = scanKingSn;
          inputValue["cashier_system_name"] = cashierSystemName;
          inputValue["bill_receipt_name"] = billReceiptName;
          inputValue["cashier_turnover_name"] = cashierTurnoverName;
          inputValue["pay_way"] = payWay;
          inputValue["amount_position"] = amountPosition;
          inputValue["receipt_picture_url"] = receiptUrl;
          inputValue["bill_first"] = billFirst;
          inputValue["intercept_always"] = interceptAlways;
          return { inputValue };
        });
      }
    } catch (error) {}
  };

  removeRelevance = (sn) => {
    const { dispatch } = this.props;
    this.setState({ relevanceListLoading: true }, () => {
      dispatch({
        type: "printDetail/unBindSubPrinter",
        payload: { sn },
        callback: () => {
          this.getRelevanceList();
        },
      });
    });
  };

  getDeviceStoreInfo = () => {
    const { match, dispatch } = this.props;
    dispatch({
      type: "printDetail/findDeviceStoreInfoBySn",
      payload: {
        sn: match.params.sn,
      },
      callback: () => {
        const { isLoading } = this.state;
        if (isLoading) {
          this.setState({ isLoading: false });
        }
      },
    });
  };

  getDeviceDetailInfo = () => {
    const { match, dispatch } = this.props;
    dispatch({
      type: "printDetail/findPrinterDetailInfoBySn",
      payload: {
        sn: match.params.sn,
      },
      callback: () => {
        const { isLoading } = this.state;
        if (isLoading) {
          this.setState({ isLoading: false });
        }
      },
    });
  };

  getDeviceStatus = () => {
    const { match, dispatch } = this.props;
    dispatch({
      type: "printDetail/getCloudPrinterCurrentState",
      payload: {
        sn: match.params.sn,
      },
    });
  };

  getRelevanceList = () => {
    const { match, dispatch } = this.props;
    dispatch({
      type: "printDetail/findPrinterAssistantInfoBySn",
      payload: {
        sn: match.params.sn,
      },
      callback: () => {
        this.setState({ relevanceListLoading: false });
      },
    });
  };

  getDeviceLog = (page = 1) => {
    const { match, dispatch } = this.props;
    dispatch({
      type: "printDetail/findDeviceEventLogBySnWithPage",
      payload: {
        sn: match.params.sn,
        page,
        page_size: 10,
      },
    });
  };

  handleDeviceLogVisible = (flag) => {
    this.setState({ deviceLogVisiable: !!flag }, () => {
      if (flag) {
        this.getDeviceLog();
      }
    });
  };

  checkIp = (value = "") => {
    const arr = value.split(".");
    if (arr.length !== 4) return false;
    for (let i = 0; i < arr.length; i++) {
      if (!/^[0-9]*$/.test(arr[i])) return false;
      if (arr[i].length > 3) return false;
    }
    return true;
  };

  lanIpChange = (e) => {
    lanIp = e.target.value;
  };

  wwanIpChange = (e) => {
    wwanIp = e.target.value;
  };

  lanIpPost = async () => {
    const {
      storeInfo: { deviceSn },
    } = this.props;
    if (!this.checkIp(lanIp)) return;
    try {
      const { code } = await updatePrinterToolInfo({
        device_sn: deviceSn,
        type: "9",
        lan_ip: lanIp,
      });
      if (code === 200) {
        this.getDeviceDetailInfo();
      }
    } catch (error) {}
  };

  wwanIpPost = async () => {
    const {
      storeInfo: { deviceSn },
    } = this.props;
    if (!this.checkIp(wwanIp)) return;
    try {
      const { code } = await updatePrinterToolInfo({
        device_sn: deviceSn,
        type: "9",
        w_wan_ip: wwanIp,
      });
      if (code === 200) {
        this.getDeviceDetailInfo();
      }
    } catch (error) {}
  };

  setInputValue = (value, key) => {
    this.setState((prevState) => {
      let inputValue = Object.assign({}, prevState.inputValue);
      inputValue[key] = value;
      return { inputValue };
    });
  };

  saveConfigure = async () => {
    let parameters = {};
    const { storeId, deviceSn } = this.props.storeInfo;
    parameters = Object.assign({}, this.state.inputValue);
    const newParams = {
      ...parameters,
      store_id: storeId,
      printer_assistant_sn: deviceSn,
    };
    const res = await updateHardPlugInfo(newParams);
    console.log("res", res);
  };

  render() {
    const { storeInfo, detailInfo, statusData, deviceLogData, relevanceList } = this.props;
    const {
      deviceLogVisiable,
      relevanceListLoading,
      isfilled,
      // inputValue: {
      //   scanKingSn,
      //   cashierSystemName,
      //   billReceiptName,
      //   cashierTurnoverName,
      //   payWay,
      //   amountPosition,
      //   receiptUrl,
      //   billFirst,
      //   interceptAlways,
      // },
    } = this.state;
    const { storeName, deviceSn, activeTime } = storeInfo;
    const isAssistant =
      deviceSn && deviceSn.startsWith("CA25") && deviceSn.length === 18 ? true : false;
    return (
      <PageHeaderLayout title="打印机详情">
        <Row gutter={{ md: 8, lg: 24, xl: 32 }}>
          {this.state.isLoading && <Spin className={styles.spinLoading} />}
          <Col xxl={18} md={18} sm={24}>
            <Card className={styles.detailCard}>
              <DescriptionList size="large" title="基本信息" style={{ marginBottom: 32 }}>
                <Description term="门店名">{storeName}</Description>
                <Description term="SN">{deviceSn}</Description>
                <Description term="激活时间">
                  {activeTime ? moment(activeTime).format("l HH:mm:ss") : ""}
                </Description>
              </DescriptionList>
              <Divider style={{ marginBottom: 32 }} />
              {!isAssistant && (
                <DescriptionList size="large" title="设备管理信息" style={{ marginBottom: 32 }}>
                  <Description term="信号强度">{detailInfo.signalStrength}</Description>
                  <Description term="通讯类型">{detailInfo.communicationType}</Description>
                  <Description term="版本">{detailInfo.firmware}</Description>
                  <Description term="ICCId">{detailInfo.simId}</Description>
                </DescriptionList>
              )}
              {isAssistant && (
                <DescriptionList size="large" title="设备管理信息" style={{ marginBottom: 32 }}>
                  <Description term="获取信息时间">
                    {detailInfo.mtime ? moment(detailInfo.mtime).format("l HH:mm:ss") : ""}
                  </Description>
                  <Description term="打印助手IP">{detailInfo.localIP}</Description>
                  <Description term="固件版本">{detailInfo.firmware}</Description>
                  <Description term="打印助手模式">{detailInfo.mode}</Description>
                  <Description term="USB设备">
                    {detailInfo.lp0Exist === "0" ? "不存在" : "存在"}
                  </Description>
                  <Description term="LAN IP">
                    <Input
                      size="small"
                      style={{ width: 120, marginRight: 10 }}
                      defaultValue={detailInfo.lanIp}
                      onChange={this.lanIpChange}
                    />
                    <Button type="primary" ghost size="small" onClick={this.lanIpPost}>
                      修改
                    </Button>
                  </Description>
                  <Description term="LAN 协议">{detailInfo.lanProto}</Description>
                  <Description term="WWAN IP">
                    <Input
                      size="small"
                      style={{ width: 120, marginRight: 10 }}
                      defaultValue={detailInfo.wwanIp}
                      onChange={this.wwanIpChange}
                    />
                    <Button type="primary" ghost size="small" onClick={this.wwanIpPost}>
                      修改
                    </Button>
                  </Description>
                  <Description term="WWAN IP协议">{detailInfo.wwanProto}</Description>
                </DescriptionList>
              )}
              <Divider style={{ marginBottom: 32 }} />
              {isAssistant && (
                <DescriptionList size="large" title="关联打印机信息" style={{ marginBottom: 32 }}>
                  <Table
                    rowKey="relevanceSn"
                    loading={relevanceListLoading}
                    pagination={false}
                    style={{ marginLeft: 16, marginRight: 16 }}
                    dataSource={relevanceList.records}
                    columns={this.columns}
                  />
                </DescriptionList>
              )}
              <Divider style={{ marginBottom: 32 }} />
              {isAssistant && (
                <DescriptionList
                  size="large"
                  title="收银机模式"
                  col={2}
                  style={{ marginBottom: 32 }}
                >
                  <p style={{ paddingLeft: 16, color: "#999" }}>绑定扫码王3（GL33）</p>
                  {items(this).map((it) => (
                    <Description term={it.name} key={it.id}>
                      <Input
                        size="small"
                        style={{ width: 150, height: 32, marginRight: 10 }}
                        value={it.value || ""}
                        onChange={(e) => this.setInputValue(e.target.value, it.id)}
                      />
                    </Description>
                  ))}
                  {options(this).map((it) => (
                    <Description term={it.name} key={it.id}>
                      <Select
                        defaultValue={it.need ? it.need : ""}
                        style={{ width: 150, textAlign: "center" }}
                      >
                        {it.optionLists.map((i) => (
                          <Option key={i.id} value={i.id}>
                            {i.name}
                          </Option>
                        ))}
                      </Select>
                    </Description>
                  ))}
                  {/* <Description term="设备号">
                    <Input
                      size="small"
                      style={{ width: 120, marginRight: 10 }}
                      value={scanKingSn || ""}
                      onChange={(e) => this.setInputValue(e.target.value, "scan_king_sn")}
                    />
                  </Description>
                   <p style={{ paddingLeft: 16, color: "#999" }}>收银系统信息配置</p> 
                  <Description term="收银系统名称">
                    <Input
                      size="small"
                      style={{ width: 120, marginRight: 10 }}
                      value={cashierSystemName}
                      onChange={(e) => this.setInputValue(e.target.value, "cashier_name")}
                    />
                  </Description>
                  <Description term="票据名称">
                    <Input
                      size="small"
                      style={{ width: 120, marginRight: 10 }}
                      value={billReceiptName}
                      onChange={(e) => this.setInputValue(e.target.value, "receipt_name")}
                    />
                  </Description>
                  <Description term="收银系统流水号名称">
                    <Input
                      size="small"
                      style={{ width: 120, marginRight: 10 }}
                      value={cashierTurnoverName}
                      onChange={(e) => this.setInputValue(e.target.value, "turnover_name")}
                    />
                  </Description>
                  <Description term="支付方式">
                    <Input
                      size="small"
                      style={{ width: 120, marginRight: 10 }}
                      value={payWay}
                      onChange={(e) => this.setInputValue(e.target.value, "payWay")}
                    />
                  </Description>
                  <Description term="获取金额位置">
                    <Input
                      size="small"
                      style={{ width: 120, marginRight: 10 }}
                      value={amountPosition}
                      onChange={(e) => this.setInputValue(e.target.value, "amount_position")}
                    />
                  </Description>
                  <Description term="小票照片上传">
                    <Input
                      size="small"
                      style={{ width: 120, marginRight: 10 }}
                      value={detailInfo.wwanIp}
                      onChange={this.setInputValue}
                    />
                  </Description> */}
                  <Description>
                    <Button
                      type="primary"
                      onClick={this.saveConfigure}
                      disabled={isfilled}
                      htmlType="submit"
                    >
                      保存
                    </Button>
                  </Description>
                </DescriptionList>
              )}
            </Card>
          </Col>
          <Col xxl={6} md={6} sm={24} span={6}>
            <Card title="当前状态" extra={<a onClick={this.getDeviceStatus}>刷新</a>}>
              <span className={styles.iconWrapper}>
                {statusData.device_state === "正常" ? (
                  <Icon type="check-circle" theme="filled" className={styles.icon} />
                ) : (
                  <Icon type="exclamation-circle" theme="filled" className={styles.icon} />
                )}

                {statusData.device_state}
              </span>
            </Card>
            <div className={styles.upgradWrapper}>
              <span className={styles.left}>设备日志</span>
              <span className={styles.right} onClick={() => this.handleDeviceLogVisible(true)}>
                显示
              </span>
            </div>
          </Col>
        </Row>
        <DeviceLogModal
          modalVisible={deviceLogVisiable}
          deviceLogData={deviceLogData}
          handleModalVisible={this.handleDeviceLogVisible}
          getDeviceLog={this.getDeviceLog}
        />
      </PageHeaderLayout>
    );
  }
}
src/containers/printManage/routes/printDetail.js
```

src/containers/printManage/printManageService.js

```js
export const updateHardPlugInfo = (params) =>
  request("api/dc/updateHardPlugInfo", {
    method: "POST",
    body: params,
  });

export const getHardPlugInfo = (params) =>
  request("api/dc/getHardPlugInfo", {
    method: "POST",
    body: params,
  });

```

```js
import React, { PureComponent } from "react";
import { connect } from "dva";
import moment from "moment";
import { Card, Row, Col, Spin, Divider, Icon, Table, Input, Button } from "antd";
import PageHeaderLayout from "../../../layouts/PageHeaderLayout";
import DescriptionList from "../../../components/DescriptionList/index";
import TableButton from "../../../components/TableButton";
import DeviceLogModal from "./deviceLogModal";
import { updatePrinterToolInfo } from "../printManageService";
import styles from "./printDetail.less";

const { Description } = DescriptionList;

let lanIp = "";
let wwanIp = "";
@connect(({ printDetail, loading }) => ({
  storeInfo: printDetail.storeInfo,
  detailInfo: printDetail.detailInfo,
  statusData: printDetail.statusData,
  deviceLogData: printDetail.deviceLogData,
  relevanceList: printDetail.relevanceList,
}))
export default class PrintDetail extends PureComponent {
  state = {
    isLoading: false,
    deviceLogVisiable: false,
    relevanceListLoading: true,
  };

  componentDidMount() {
    const { match } = this.props;
    this.setState({ isLoading: true }, () => {
      this.getDeviceStoreInfo();
      this.getDeviceDetailInfo();
      this.getDeviceStatus();
      if (match.params.sn.startsWith("CA25") && match.params.sn.length === 18) {
        this.getRelevanceList();
      }
    });
  }

  columns = [
    { title: "设备号", dataIndex: "relevanceSn" },
    {
      title: "打印机类型",
      dataIndex: "printerType",
      render: (type) => <span>{type === "1" ? "USB打印机" : "网络打印机"}</span>,
    },
    {
      title: "IP地址",
      dataIndex: "localIP",
      render: (text) => <span>{text ? text.split(":")[0] : "--"}</span>,
    },
    { title: "品牌", dataIndex: "deviceProvider" },
    {
      title: "纸宽",
      dataIndex: "paperSpec",
      render: (text) => <span>{text + "mm"}</span>,
    },
    {
      title: "解绑打印机",
      dataIndex: "remove",
      render: (text, record) => (
        <TableButton type="primary" onClick={() => this.removeRelevance(record.relevanceSn)}>
          删除
        </TableButton>
      ),
    },
  ];

  removeRelevance = (sn) => {
    const { dispatch } = this.props;
    this.setState({ relevanceListLoading: true }, () => {
      dispatch({
        type: "printDetail/unBindSubPrinter",
        payload: { sn },
        callback: () => {
          this.getRelevanceList();
        },
      });
    });
  };

  getDeviceStoreInfo = () => {
    const { match, dispatch } = this.props;
    dispatch({
      type: "printDetail/findDeviceStoreInfoBySn",
      payload: {
        sn: match.params.sn,
      },
      callback: () => {
        const { isLoading } = this.state;
        if (isLoading) {
          this.setState({ isLoading: false });
        }
      },
    });
  };

  getDeviceDetailInfo = () => {
    const { match, dispatch } = this.props;
    dispatch({
      type: "printDetail/findPrinterDetailInfoBySn",
      payload: {
        sn: match.params.sn,
      },
      callback: () => {
        const { isLoading } = this.state;
        if (isLoading) {
          this.setState({ isLoading: false });
        }
      },
    });
  };

  getDeviceStatus = () => {
    const { match, dispatch } = this.props;
    dispatch({
      type: "printDetail/getCloudPrinterCurrentState",
      payload: {
        sn: match.params.sn,
      },
    });
  };

  getRelevanceList = () => {
    const { match, dispatch } = this.props;
    dispatch({
      type: "printDetail/findPrinterAssistantInfoBySn",
      payload: {
        sn: match.params.sn,
      },
      callback: () => {
        this.setState({ relevanceListLoading: false });
      },
    });
  };

  getDeviceLog = (page = 1) => {
    const { match, dispatch } = this.props;
    dispatch({
      type: "printDetail/findDeviceEventLogBySnWithPage",
      payload: {
        sn: match.params.sn,
        page,
        page_size: 10,
      },
    });
  };

  handleDeviceLogVisible = (flag) => {
    this.setState({ deviceLogVisiable: !!flag }, () => {
      if (flag) {
        this.getDeviceLog();
      }
    });
  };

  checkIp = (value = "") => {
    const arr = value.split(".");
    if (arr.length !== 4) return false;
    for (let i = 0; i < arr.length; i++) {
      if (!/^[0-9]*$/.test(arr[i])) return false;
      if (arr[i].length > 3) return false;
    }
    return true;
  };

  lanIpChange = (e) => {
    lanIp = e.target.value;
  };

  wwanIpChange = (e) => {
    wwanIp = e.target.value;
  };

  lanIpPost = async () => {
    const {
      storeInfo: { deviceSn },
    } = this.props;
    if (!this.checkIp(lanIp)) return;
    try {
      const { code } = await updatePrinterToolInfo({
        device_sn: deviceSn,
        type: "9",
        lan_ip: lanIp,
      });
      if (code === 200) {
        this.getDeviceDetailInfo();
      }
    } catch (error) {}
  };

  wwanIpPost = async () => {
    const {
      storeInfo: { deviceSn },
    } = this.props;
    if (!this.checkIp(wwanIp)) return;
    try {
      const { code } = await updatePrinterToolInfo({
        device_sn: deviceSn,
        type: "9",
        w_wan_ip: wwanIp,
      });
      if (code === 200) {
        this.getDeviceDetailInfo();
      }
    } catch (error) {}
  };

  render() {
    const { storeInfo, detailInfo, statusData, deviceLogData, relevanceList } = this.props;
    const { deviceLogVisiable, relevanceListLoading } = this.state;
    const { storeName, deviceSn, activeTime } = storeInfo;
    const isAssistant =
      deviceSn && deviceSn.startsWith("CA25") && deviceSn.length === 18 ? true : false;
    return (
      <PageHeaderLayout title="打印机详情">
        <Row gutter={{ md: 8, lg: 24, xl: 32 }}>
          {this.state.isLoading && <Spin className={styles.spinLoading} />}
          <Col xxl={18} md={18} sm={24}>
            <Card className={styles.detailCard}>
              <DescriptionList size="large" title="基本信息" style={{ marginBottom: 32 }}>
                <Description term="门店名">{storeName}</Description>
                <Description term="SN">{deviceSn}</Description>
                <Description term="激活时间">
                  {activeTime ? moment(activeTime).format("l HH:mm:ss") : ""}
                </Description>
              </DescriptionList>
              <Divider style={{ marginBottom: 32 }} />
              {!isAssistant && (
                <DescriptionList size="large" title="设备管理信息" style={{ marginBottom: 32 }}>
                  <Description term="信号强度">{detailInfo.signalStrength}</Description>
                  <Description term="通讯类型">{detailInfo.communicationType}</Description>
                  <Description term="版本">{detailInfo.firmware}</Description>
                  <Description term="ICCId">{detailInfo.simId}</Description>
                </DescriptionList>
              )}
              {isAssistant && (
                <DescriptionList size="large" title="设备管理信息" style={{ marginBottom: 32 }}>
                  <Description term="获取信息时间">
                    {detailInfo.mtime ? moment(detailInfo.mtime).format("l HH:mm:ss") : ""}
                  </Description>
                  <Description term="打印助手IP">{detailInfo.localIP}</Description>
                  <Description term="固件版本">{detailInfo.firmware}</Description>
                  <Description term="打印助手模式">{detailInfo.mode}</Description>
                  <Description term="USB设备">
                    {detailInfo.lp0Exist === "0" ? "不存在" : "存在"}
                  </Description>
                  <Description term="LAN IP">
                    <Input
                      size="small"
                      style={{ width: 120, marginRight: 10 }}
                      defaultValue={detailInfo.lanIp}
                      onChange={this.lanIpChange}
                    />
                    <Button type="primary" ghost size="small" onClick={this.lanIpPost}>
                      修改
                    </Button>
                  </Description>
                  <Description term="LAN 协议">{detailInfo.lanProto}</Description>
                  <Description term="WWAN IP">
                    <Input
                      size="small"
                      style={{ width: 120, marginRight: 10 }}
                      defaultValue={detailInfo.wwanIp}
                      onChange={this.wwanIpChange}
                    />
                    <Button type="primary" ghost size="small" onClick={this.wwanIpPost}>
                      修改
                    </Button>
                  </Description>
                  <Description term="WWAN IP协议">{detailInfo.wwanProto}</Description>
                </DescriptionList>
              )}
              <Divider style={{ marginBottom: 32 }} />
              {isAssistant && (
                <DescriptionList size="large" title="关联打印机信息" style={{ marginBottom: 32 }}>
                  <Table
                    rowKey="relevanceSn"
                    loading={relevanceListLoading}
                    pagination={false}
                    style={{ marginLeft: 16, marginRight: 16 }}
                    dataSource={relevanceList.records}
                    columns={this.columns}
                  />
                </DescriptionList>
              )}
            </Card>
          </Col>
          <Col xxl={6} md={6} sm={24} span={6}>
            <Card title="当前状态" extra={<a onClick={this.getDeviceStatus}>刷新</a>}>
              <span className={styles.iconWrapper}>
                {statusData.device_state === "正常" ? (
                  <Icon type="check-circle" theme="filled" className={styles.icon} />
                ) : (
                  <Icon type="exclamation-circle" theme="filled" className={styles.icon} />
                )}

                {statusData.device_state}
              </span>
            </Card>
            <div className={styles.upgradWrapper}>
              <span className={styles.left}>设备日志</span>
              <span className={styles.right} onClick={() => this.handleDeviceLogVisible(true)}>
                显示
              </span>
            </div>
          </Col>
        </Row>
        <DeviceLogModal
          modalVisible={deviceLogVisiable}
          deviceLogData={deviceLogData}
          handleModalVisible={this.handleDeviceLogVisible}
          getDeviceLog={this.getDeviceLog}
        />
      </PageHeaderLayout>
    );
  }
}
jiude
```

