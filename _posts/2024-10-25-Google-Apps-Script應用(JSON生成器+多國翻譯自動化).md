---
layout: post
title: Google Apps Script應用(JSON生成器+多國翻譯自動化)
date: 2024-10-25 17:09 +0800
description:
image:
category: [Frontend]
tags: [Vue, JS]
published: true
sitemap: false
---
### 前言
對於網頁或是APP，常會有閱讀非母語的時候，有些人會選擇透過網頁翻譯的方式直接閱讀，這會導致翻譯不到位、語意偏移等等問題，所以在比較完善的系統上，開發者會添加所謂的**多國語言**，讓使用者閱讀起來更加便利，而今天來分享一下簡單的多國語言的管理方法。

### 翻譯方法
若是就目前的開發方式，普遍會找尋i18n套件，而有些也套件支援AI輔助翻譯，讓語意更貼近開發者想表達的內容，但牽扯到效能及其他成本的考量，目前還是建議透過人工來進行翻譯最為妥當，但是對於翻譯這樣的工作，有時候不會是工程師來填寫，可能會由PM或是熟知他國語言者來完成，那我們目標會是如何打造一個舒適的分工環境。

### 使用工具
* [Google試算表](https://docs.google.com/spreadsheets/create?hl=zh-tw) - Google 對於Excel的雲端服務
* [Google Apps Script](https://developers.google.com/apps-script?hl=zh-tw) - 透過JavaScript撰寫Google的簡易腳本。

### 實作目標
* 需要一個地方，用來管理翻譯的所以內容，而目前在I18n的套件上，主要支援**XML**、**YAML**、**JSON**這三種檔案格式，而本次使用JSON(套件也使用JSON的關係)，我們要從Google試算表匯出JSON檔後，再由這個JSON檔去產生相對應的翻譯檔去做翻譯。
![](/assets/img/post/2024-1025/p1.png)

### Google試算表
![](/assets/img/post/2024-1025/p0.png)
依據定義好的編輯方式，這邊先簡單定義一下excel。

### 開始編輯Google Apps Script
* 首先要打開**擴充功能** &rarr; **Apps Script**來做腳本的編輯，這邊直接附上Code，簡單說就是將Excel的內容轉符合理想的JSON格式，當中有用到[Sheet Class](https://developers.google.com/apps-script/reference/spreadsheet/sheet?hl=zh-tw)，他有提供許多會取表格內容的功能。

```js
  var output = { "locale": ["zh-TW", "en-US", "ja-JP", "zh-CN", "dev"] }
  const Lang = { "dev-TW": 2, "zh-TW": 6, "en-US": 7, "ja-JP":10 , "zh-CN":9, "dev":2}
  const Rx = "A3:K"
  const Sheets = SpreadsheetApp.getActiveSpreadsheet();
  const page = {};
  const bkPage = {}


  const devMode = () => JsonExport('dev');
  const normalMode = () => JsonExport('normal');

  function JsonExport(mode) {
    sheetsData(Sheets.getSheets());
    const list = Object.keys(page);
    list.forEach(k => {
      page[k].forEach((row, i) => {
        let ref = output;
        const p = i + 3;
        if (/#/.test(row[0])) return;
        const zh = row[Lang['zh-TW']];
        const en = row[Lang['en-US']];
        const jp = row[Lang['ja-JP']].split("_")[0] || row[Lang['en-US']];
        const cn = row[Lang['zh-CN']];
        const JsonPath = row[Lang[mode]]
        const values = [zh, en, jp, cn];
        if (!values.join('')) return;
        values.push(`${k}/${p}`)
        const keys = JsonPath.split('.');

        const keyLst = keys.length - 1;
        keys.forEach((key, i) => {
          if (i === keyLst) return ref[key] = values;
          if (!ref[key]) ref[key] = {};
          ref = ref[key];
        })
      })
    })
    displayJson(JSON.stringify(output, null, 2));
  }

  const sheetsData = (Sheets) => {
    const sheetLen = Sheets.length;
    for (let i = 0; i < sheetLen; i++) {
      const sheetName = Sheets[i].getName();
      const lstRow = Sheets[i].getLastRow();
      const rng = Rx + lstRow;
      const vals = Sheets[i].getRange(rng).getValues();
      page[sheetName] = vals;
    }
  }

  function insertBreakPoint(text) {
    // 逸脫
    text = text.replace(/\\\\/g, "\\");
    text = text.replace(/\/\//g, "\\/\\/");
    // for排版
    text = text.replace(/\[\s*/g, "[").replace(/\s*\]/g, "]").replace(/\"\,\s*\"/g, "\", \"");
    return text;
  }

  function displayJson(content) {
    const out = insertBreakPoint(content)
    const ori = `<textarea style='width:80%;' rows='35'>${out}</textarea>`;
    const html = HtmlService.createHtmlOutput(ori);
    html.setWidth(1600)
    html.setHeight(600);
    SpreadsheetApp.getUi().showModalDialog(html, 'Exported JSON');
  }
```

* 當我們執行devMode後，可以回到試算表那一頁，看看是不是有正確的JSON內容。
![](/assets/img/post/2024-1025/p3.png)

### 觀察目前專案的多國套件

```js
  import Vue from 'vue';
  import VueI18n from 'vue-i18n';
  import cookie from 'js-cookie';
  /// 各種翻譯檔
  import zh_tw from './locales/zh-TW.json'; 
  import en_us from './locales/en-US.json';
  import ja_jp from './locales/ja-JP.json';
  import zh_cn from './locales/zh-CN.json';
  import dev_tw from './locales/dev.json';

  Vue.use(VueI18n);

  let locale = '';
  function setLanguage() {
    locale = navigator.language || navigator.userLanguage;
    if (cookie.get('currentLang')) {
      locale = cookie.get('currentLang');
    } else {
      cookie.set('currentLang', locale);
    }

    switch (locale) {
      case 'zh_tw':
        locale = 'zh_tw';
        break;
      case 'en_us':
        locale = 'en_us';
        break;
      case 'ja_jp':
        locale = 'ja_jp';
        break;
      case 'dev_tw':
        locale = 'dev_tw';
        break;
      default:
        locale = 'zh_tw';
        cookie.set('currentLang', locale);
        break;
    }
  }

  setLanguage();

  const messages = {
    zh_tw, en_us, ja_jp, zh_cn, dev_tw
  };

  // Create VueI18n instance with options
  const i18n = new VueI18n({
    silentTranslationWarn: true,
    locale,
    messages, // set locale messages
  });

  export default i18n;
```

就這個Vue專案來說，他是使用了vue-i18n來當作多國套件，從Code來看，當專案執行時，會先找到目前的語言去導入所要的翻譯檔，接著我們要將試算表所輸出的JSON來產生對應的翻譯檔。

### 執行專案
* 當開啟前端專案的流程中，多加一個產生翻譯檔的流程，為了確保每次翻譯檔都是最新的，所以每次run專案時都會跑這項流程。
![](/assets/img/post/2024-1025/p4.png)
* 這邊附上js的內容。

```js
const fs = require('fs');

function convertHandler(target, keyName = '', index, originData) {
  switch (typeof target) {
    case 'object': {
      // handle array value
      if (Array.isArray(target)) {
        originData[keyName] = originData[keyName][index];
        return;  
      }

      // handle object tree
      for (const key in target) {
        if (target.hasOwnProperty(key)) {
          if (Array.isArray(target[key])) {
            target[key] = target[key][index];
          }
          convertHandler(target[key], key, index);
        }
      }
      return;
    }
    default:
      return;
  }
}

// check path is exist
const pathIsExist = fs.existsSync("./src/lang/locales");
if (!pathIsExist) {
  fs.mkdir("./src/lang/locales",function(err){
    if (err) {
      return console.error(err);
    }
  });
}

const originSouce = fs.readFileSync('./src/lang/locales.json', 'utf-8');
const source = JSON.parse(originSouce);
if (!source.hasOwnProperty('locale')) {
  throw new Error('locale key is required')
}

for (let i = 0; i < source.locale.length; i++) {
  const localeObj = JSON.parse(originSouce);
  for (const key in localeObj) {
    convertHandler(localeObj[key], key, i, localeObj);
  }

  fs.writeFileSync(`./src/lang/locales/${source.locale[i]}.json`, JSON.stringify(localeObj));
}

```

* 執行完後去檢查路徑是否產生正確的JSON檔。

![](/assets/img/post/2024-1025/p5.png)

### 結語
這項功能掩飾，除了解決前端工程師，在每次翻譯都要根據特定語言，對每個翻譯檔進行逐一修改之外，也可以認識一下如何應用Google Apps Script，讓繁雜的程序自動化。







