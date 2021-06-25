# 下載報表

### 提供把 token 當 quey 傳的報表

用 article element 請求即可。

```html
<a href="https://xxx.com/report/?=token=xr34dr2&foo=123"> 下載報表 </a>
```

### 不提供把 token 當 quey 傳的報表

這個情況比較尷尬，透過從公司的前端老司機和自己餵狗得到了稍微偏門但行得通的方法，至少可以確保請求不會因為沒帶 token 被擋下。<br />

這邊留意一下範例的 `data` 已經被我改過變成有 `blob` (報表) 跟 `filename` (檔名)屬性的格式，是因為要解決檔案標題不見才這麼做的，我晚點會繼續說明。<br />

如果你只是要解決無法帶上 token 的問題，直接用 `blob` 做處理就好。

```javascript
/**
 * 下載不提供 token 當 query 的報表
 * @param {Object} instance - axios實例
 * @param {string} url - 報表服務網址
 * @param {Object} params - 報表參數
 */
function downloadFile({ instance, url, params = {} }) {
  try {
    // 請求 Blob 格式的資料
    const data = instance.get(url, {
      params,
      responseType: "blob",
    });

    // 轉成 CSV
    const url = window.URL.createObjectURL(
      new Blob([data.blob], { type: "text/csv" })
    );

    // 塞進 article element 裡
    const link = document.createElement("a");
    link.style.display = "none";
    link.href = url;
    link.setAttribute("download", data.filename || "file");

    // 觸發點擊下載後銷毀
    document.body.appendChild(link);
    link.click();
    window.URL.revokeObjectURL(url);
    document.body.removeChild(link);
  } catch (_) {
    console.error("Download file failed !");
  }
}
```

### 檔案標題遺失

如果是用前一個範例單純打 XHR 請求完再把 Blob 轉成 CSV，原始檔案的標題會遺失。之前還太菜就暫時用硬拼的方式組標題，不過現在後端很需要我的協助，該還的債總是要還。

### 關於 Headers

待捕

### 解決方式

請後端協助在 `Access-Control-Expose-Headers` 補上 `Content-Disposition`，這樣前端 get Header 的屬性才不會被 CORS 擋。<br />

判斷是在下載報表的請求類型的話，從 `Content-Disposition` 中解析 filename 並組成符合 `downloadFile` 的格式。<br />

Axios 範例：

```javascript
const instance = axios.create();
// ...
instance.interceptors.response.use((response) => {
  if (response.headers["content-type"] === "text/csv") {
    let filename;
    try {
      filename = [
        (str) => response.request.getResponseHeader(str),
        decodeURI,
        (str) => str.match(/filename\*=UTF-8''(.*)/),
        (arr) => (arr.length ? arr[1] : null),
      ].reduce((result, cb) => cb(result), "Content-Disposition");
    } catch (_) {}
    return {
      blob: response.data,
      filename,
    };
  }
  // ...
});
```

下載報表範例：

```javascript
downloadFile({
  instance: fooAxiosInstance,
  url: "https://xxx.com/report/",
  params: {
    foo: 123,
  },
});
```

### 參考

- [MDN - Access-Control-Expose-Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Expose-Headers)
- [MDN - Content-Disposition](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Disposition)
- [Stack Overflow - Cross Domain Resource Sharing GET: 'refused to get unsafe header “etag”' from Response](https://stackoverflow.com/questions/5822985/cross-domain-resource-sharing-get-refused-to-get-unsafe-header-etag-from-re)

###### tags: `Work` `JavaScript`
