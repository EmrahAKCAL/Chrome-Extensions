<h2>manifest.json</h2>

````

{
  "manifest_version": 3, // manifest dosyasının versiyonu (3) 
  "name": "Tabs Manager", // uzantının adı
  "version": "1.0", // uzantının versiyonu
  "description": "Tab Manager for Chrome Dev Docs", // uzantının açıklaması
  "icons": { // uzantının simgeleri (16, 32, 48, 128)
    "16": "images/icons/icon-16.png",
    "32": "images/icons/icon-32.png",
    "48": "images/icons/icon-48.png",
    "128": "images/icons/icon-128.png"
  },
  "action": { // uzantının butonu (popup.html)
    "default_popup": "popup/popup.html" // popup.html dosyası (varsayılan)
  },
  "host_permissions": [ // uzantının erişebileceği siteler
    "https://developer.chrome.com/*" // developer.chrome.com/* (tüm sayfalar)
  ],
  "permissions": [ // uzantının erişebileceği özellikler
    "tabGroups" // tabGroups özelliği (tab grupları)
  ]

}


````


<h2>popup.js</h2>

```

//// bu sayfaları aç ama diğerlerini açma demek istiyoruz burada (örneğin https://developer.chrome.com/docs/extensions/*)
const tabs = await chrome.tabs.query({
    url: [
        "https://developer.chrome.com/docs/webstore/*",
        "https://developer.chrome.com/docs/extensions/*",
    ]
});

const collator = new Intl.Collator(); //// alfabetik sıralama için kullanıyoruz
tabs.sort((a,b) => collator.compare(a.title, b.title)); //// sekmelerin başlıklarına göre sıralıyoruz
const template = document.getElementById("li_template"); //// <template> elementini alıyoruz
const elements = new Set(); //// <li> elementlerini tutmak için bir Set oluşturuyoruz (aynı elemanı birden fazla kez eklememek için)
for(const tab of tabs) { //// sekmeleri tek tek dolaşıyoruz
    const element = template.content.firstElementChild.firstElementChild.cloneNode(true); //// <template> elementinin içindeki <li> elementini klonluyoruz (true parametresi ile içindeki tüm alt elemanları da klonluyoruz) ve klonu alıyoruz (cloneNode() metodunun döndürdüğü değer klonlanan element)
    const title = tab.title.split(" - ")[0].trim(); //// sekmelerin başlıklarını alıyoruz (örneğin "chrome.tabs - Google Chrome") ve gereksiz kısımları atıyoruz
    const pathname = new URL(tab.url).pathname.slice("/docs".length); //// sekmelerin URL'lerini alıyoruz (örneğin "https://developer.chrome.com/docs/extensions/reference/tabs/") ve gereksiz kısımları atıyoruz (örneğin "/docs")
    element.querySelector(".title").textContent = title; //// <li> elementinin içindeki <span class="title"> elementinin içine başlığı yazıyoruz
    element.querySelector(".pathname").textContent = pathname; //// <li> elementinin içindeki <span class="pathname"> elementinin içine URL'nin gereksiz kısımlarını atıp kalanını yazıyoruz 
    const a = element.querySelector("a"); //// <li> elementinin içindeki <a> elementini alıyoruz (sekmeyi açmak için)
    if (a) { //// <a> elementi varsa (yani sekmeyi açmak için tıklayabileceğimiz bir element varsa)
        a.addEventListener("click", async () => { //// <a> elementine tıklandığında sekmeyi açmak için gerekli kodu yazıyoruz (async fonksiyon kullanıyoruz çünkü await kullanacağız)
            await chrome.tabs.update(tab.id, { active: true }); //// sekmeyi aktif hale getiriyoruz (yani sekmeye tıklıyoruz)
            await chrome.windows.update(tab.windowId, { focused: true }); //// sekmeyi içeren pencereyi aktif hale getiriyoruz (yani pencereye tıklıyoruz)
        });
    }
    elements.add(element); //// <li> elementini Set'e ekliyoruz (aynı elemanı birden fazla kez eklememek için)
}
document.querySelector("ul").append(...elements); //// <ul> elementinin içine <li> elementlerini ekliyoruz (spread operator kullanıyoruz çünkü append() metoduna birden fazla parametre verebiliyoruz)

const button = document.querySelector("button"); //// <button> elementini alıyoruz (sekmeleri gruplamak için)
button.addEventListener("click", async () => { //// <button> elementine tıklandığında sekmeleri gruplamak için gerekli kodu yazıyoruz (async fonksiyon kullanıyoruz çünkü await kullanacağız)
    const tabIds = tabs.map(({id}) => id); //// sekmelerin ID'lerini alıyoruz
    const group = await chrome.tabs.group({tabIds}); //// sekmeleri grupluyoruz (grup ID'sini alıyoruz)
    await chrome.tabGroups.update(group, {title: "Chrome Docs"}); //// grup başlığını değiştiriyoruz (grup ID'sini ve yeni başlığı alıyoruz)
});



```
