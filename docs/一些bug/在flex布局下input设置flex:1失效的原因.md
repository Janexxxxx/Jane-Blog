![image-20220315222354249](/Users/xujing/Library/Application Support/typora-user-images/image-20220315222354249.png)

![image-20220315222554707](/Users/xujing/Library/Application Support/typora-user-images/image-20220315222554707.png)

如图，点击输入框输入文字时，会跳出“取消”，输入框需要动态缩小，相当于两栏布局，input框动态改变大小，右边的“取消”固定margin-left设置间隔距离。

出现问题：flex:1失效，input不会动态改变大小，在安卓和ios都正常，pos机出现问题。

解决参考：[在flex布局下input设置flex:1失效的原因](https://blog.csdn.net/weixin_43130844/article/details/100123931?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~aggregatepage~first_rank_ecpm_v1~rank_v31_ecpm-2-100123931.pc_agg_new_rank&utm_term=flex+%E5%AE%89%E5%8D%93%E6%89%8B%E6%9C%BA%E5%A4%B1%E6%95%88+%E5%B8%83%E5%B1%80&spm=1000.2123.3001.4430)

```js
//原代码js
<div className={styles.sqbAutoSearch}>
        <div className={styles.content}>
          <img src={search} />
          <input
            ref={this.lv}
            autoComplete="off"
            onBlur={this.blur}
            onFocus={this.focus}
            value={value}
            onChange={this.onSearch}
            onCompositionStart={this.compositionStart}
            onCompositionEnd={this.compositionEnd}
            type="search"
            placeholder="请输入搜索内容"
          />
        </div>
        {value ? (
          <div className={styles.cancel} onClick={this.onCancel}>
            取消
          </div>
        ) : (
          ''
        )}
      </div>

//css
//出错原因：input中又设置flex:1。
//改：去掉input中flex:1，设置input宽度为100%。
.content {
  flex: 1;
  background: #f6f6f6;
  display: flex;
  align-items: center;
  height: 60px /* 60/75 */;
  border-radius: 8px /* 8/75 */;
  img {
    height: 34px /* 34/75 */;
    width: 34px /* 34/75 */;
    margin-left: 16px /* 16/75 */;
  }
  input {
    width: 100%;
    font-size: 30px /* 30/75 */;
    background: #f6f6f6;
    border: none;
    outline: none;
    height: 60px;
    line-height: 60px;
    padding: 0;
    vertical-align: middle;
    // flex: 1;
    margin-left: 16px /* 16/75 */;
    -webkit-appearance: none;
    &::-webkit-search-cancel-button {
      display: none;
    }
  }
}
.cancel {
  margin-left: 24px /* 24/75 */;
  color: #d9af5c;
  font-size: 30px /* 30/75 */;
}
```

