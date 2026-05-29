# WebSocket Hands-on Lab - 即時共編繪圖網頁

今天這堂課，我們會一步一步從 0 開始打造一個支援多人連線同步的繪圖網頁：

讓我們開始吧＾-＾

---

## 工商時間 >_<

有問題想詢問或想法討論歡迎加我的 IG (easonlu0303)。

---

## 今天會用到的技術

* HTML, CSS, JavaScript
* Canvas API
* WebSocket & Socket.io
* Python
* Replit

---

## 今天成果

功能：

* 使用滑鼠畫圖
* 多人同時畫並且即時同步
* 可以改顏色和筆刷大小
* 清空畫布

## 專案結構

建立以下結構：

```
.
├── server.py
├── requirements.txt
└── public
    ├── index.html
    └── index.js
```

---

## Step 1 安裝套件

開啟：

```
requirements.txt
```

內容：

```
python-socketio
eventlet
```

執行：

```bash
pip install -r "requirements.txt"
```

> Q: requirements.txt 是什麼？
> A: python 程式碼套件清單，記錄需安裝的函式庫依賴。 

## Step 2 最基本網頁架構

開啟：

```
public/index.html
```

> Q: HTML 是什麼？
> A: HTML 作為網頁的骨架，所有基礎元件（例如按鈕，文字，圖片，畫布，都是 HTML 元件。 

### 構成畫布 (canvas) 的基本程式碼

```html id="oifnsz"
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1.0"
    />
    <title>Realtime Draw</title>
  </head>

  <body>
    <canvas id="c"></canvas>
  </body>

  <script src="./index.js"></script>
</html>
```

### canvas 是什麼？

```html
<canvas id="c"></canvas>
```

Canvas 是 HTML 中管理畫布的元素，可以透過前端 JavaScript 程式實現：

* 畫圖
* 做遊戲
* 做動畫
* 做白板等功能

## Step 3 白板繪圖功能＆邏輯

開啟：

```text id="4qfc8u"
public/index.js
```

### 取得 Canvas & ctx 物件

```js
const canvas =
  document.getElementById("c");

const ctx =
  canvas.getContext("2d");
```

### 修正 Canvas 大小

加入：

```js id="6z7bd0"
function resize() {
  canvas.width =
    window.innerWidth;

  canvas.height =
    window.innerHeight;
}

resize();

window.addEventListener(
  "resize",
  resize
);
```

### 實作畫圖功能

1. 建立狀態

```js id="fw8u2u"
let drawing = false;
let current = {
  x: 0,
  y: 0,
};
```

2. 滑鼠按下

```js id="z7j3bk"
canvas.addEventListener(
  "mousedown",
  (e) => {
    drawing = true;
    current.x = e.clientX;
    current.y = e.clientY;
  }
);
```

3. 滑鼠放開

```js 
window.addEventListener(
  "mouseup",
  () => {
    drawing = false;
  }
);
```

4. 滑鼠移動

```js
canvas.addEventListener(
  "mousemove",
  (e) => {
    if (!drawing) return;

    const x = e.clientX;
    const y = e.clientY;

    drawLine(
      current.x,
      current.y,
      x,
      y
    );

    current.x = x;
    current.y = y;
  }
);
```

5. 建立 drawLine

```js id="9v2bh0"
function drawLine(
  x1,
  y1,
  x2,
  y2
) {
  ctx.beginPath(); // 開始新的路徑
  ctx.moveTo(x1, y1); // 移動畫筆位置
  ctx.lineTo(x2, y2); // 畫到指定位置
  ctx.stroke(); // 真正把線畫出來
}
```

## Step 5 美化筆刷

修改 drawLine：

```js id="jlwm1l"
function drawLine(
  x1,
  y1,
  x2,
  y2
) {
  ctx.strokeStyle = "#111111"; // 畫筆顏色
  ctx.lineWidth = 4; // 畫筆寬度
  ctx.lineCap = "round"; // 線條尾端變圓滑
  ctx.lineJoin = "round"; // 線條轉角變圓滑

  ctx.beginPath();
  ctx.moveTo(x1, y1);
  ...
}
```
## Step 6 建立 WebSocket Server

建立：

```text
server.py
```

> Q: WebSocket 是什麼？
> A: WebSocket 是一種「持續連線」的即時通訊技術，可以讓 Client 和 Server 雙向即時傳輸資料。
> 一般 HTTP 是 Request -> Response 結束，但 WebSocket 會維持連線不中斷。

---

### 建立 WebSocket Server

```py
import socketio
import eventlet

sio = socketio.Server(
    cors_allowed_origins="*"
)

app = socketio.WSGIApp(
    sio,
    static_files={
        "/": "public/index.html",
        "/index.js": "public/index.js",
    }
)

@sio.event
def connect(sid, environ):
    print("connected:", sid)

@sio.event
def disconnect(sid):
    print("disconnect:", sid)

print("server running")

eventlet.wsgi.server(
    eventlet.listen(("0.0.0.0", 5000)),
    app
)
```

### socketio.Server()

```py
sio = socketio.Server()
```

建立 Socket.io WebSocket Server。

這個物件負責：

* 接收 Client 連線
* 收送事件
* 廣播資料
* 管理所有使用者

---

### connect event

```py
@sio.event
def connect(sid, environ):
    print("connected:", sid)
```

當有使用者連線時觸發。



## Step 7 執行前端程式碼

在 Replit Shell 輸入：

```bash
python server.py
```

看看在預覽頁面看看會發生什麼事？

---

## Step 8 前端連線 WebSocket

回到：

```text
public/index.html
```

加入：

```html
<script src="https://cdn.socket.io/4.8.1/socket.io.min.js"></script>
```

放在：

```html
<script src="./index.js"></script>
```

上面。

---

### Socket.io Client

這行的作用：

```html
<script src="https://cdn.socket.io/4.8.1/socket.io.min.js"></script>
```

是載入 Socket.io 前端函式庫。

這樣前端 JS 腳本才能建立 WebSocket 連線。

---

## Step 9 建立 WebSocket 連線

回到：

```text
public/index.js
```

加入：

```js
const ws = io();
```

> Q: io() 是什麼？
> A: io() 會自動連線到目前網站的 WebSocket Server。

---

### 測試連線

加入：

```js
ws.on("connect", () => {
  console.log("connected");
});
```

---

### ws.on()

```js
ws.on(...)
```

代表：

# 監聽事件

例如：

```js
ws.on("draw")
```

代表：

「當收到 draw 事件時執行程式」。

---

### 開啟 DevTools

打開瀏覽器 Console。

如果成功應該會看到：

```text
connected
```

Server 也會看到：

```text
connected: xxxxx
```

---

## Step 10 發送畫圖事件

現在我們已經有 WebSocket 連線。

接下來：

# 把畫圖資料送到 Server

---

### 建立 emitDraw

加入：

```js
function emitDraw(
  x1,
  y1,
  x2,
  y2
) {
  ws.emit("draw", {
    from: {
      x: x1,
      y: y1,
    },

    to: {
      x: x2,
      y: y2,
    },
  });
}
```

---

### emit()

```js
ws.emit(...)
```

代表：

# 發送事件到 Server

格式：

```js
ws.emit("事件名稱", 資料)
```

---

### draw event

這裡送出的資料：

```js
ws.emit("draw", ...)
```

代表：

# 「有人正在畫圖」

---

### 修改 mousemove

找到：

```js
canvas.addEventListener(
  "mousemove",
```

加入：

```js
emitDraw(
  current.x,
  current.y,
  x,
  y
);
```

完整：

```js
canvas.addEventListener(
  "mousemove",
  (e) => {
    if (!drawing) return;

    const x = e.clientX;
    const y = e.clientY;

    drawLine(
      current.x,
      current.y,
      x,
      y
    );

    emitDraw(
      current.x,
      current.y,
      x,
      y
    );

    current.x = x;
    current.y = y;
  }
);
```

---

## Step 11 Server 接收畫圖事件

回到：

```text
server.py
```

加入：

```py
@sio.on("draw")
def draw(sid, data):
    sio.emit(
        "draw",
        data,
        skip_sid=sid
    )
```

---

### @sio.on("draw")

代表：

# 當 Server 收到 draw 事件時

執行下面程式。

---

### data 是什麼？

data 就是前端 emit 過來的資料：

```js
{
  from: {...},
  to: {...}
}
```

---

### sio.emit()

```py
sio.emit(...)
```

代表：

# Server 廣播資料

送給所有連線中的 Client。

---

### skip_sid=sid

```py
skip_sid=sid
```

意思：

# 不要送回原本的人

因為自己本來就已經畫過了。

---

## Step 12 接收別人的畫圖

回到：

```text
public/index.js
```

加入：

```js
ws.on("draw", (data) => {
  drawLine(
    data.from.x,
    data.from.y,
    data.to.x,
    data.to.y
  );
});
```

---

### 現在發生什麼事？

流程：

```text
玩家A畫圖
    ↓
emit("draw")
    ↓
Server收到
    ↓
Server廣播
    ↓
玩家B收到
    ↓
drawLine()
```

---

## Step 13 測試多人同步

複製 Replit 網址。

例如：

```text
https://xxxx.replit.app
```

---

### 多開幾個分頁

現在應該可以看到：

# 即時同步畫圖

---

### 這其實就是：

* Discord 即時聊天
* 線上遊戲同步
* Google Docs
* Figma
* 多人白板

背後的核心概念。

---

## Step 14 加入顏色功能

建立狀態：

```js
let color = "#111111";
```

---

### 建立工具列

加入：

```js
const toolbar =
  document.createElement("div");

document.body.appendChild(toolbar);
```

---

### 建立 color picker

```js
const colorInput =
  document.createElement("input");

colorInput.type = "color";

colorInput.value = color;

colorInput.oninput = (e) => {
  color = e.target.value;
};

toolbar.appendChild(colorInput);
```

---

### input type="color"

HTML 內建：

# 顏色選擇器

---

### 修改 drawLine

```js
function drawLine(
  x1,
  y1,
  x2,
  y2,
  color
) {
  ctx.strokeStyle = color;

  ctx.lineWidth = 4;

  ctx.lineCap = "round";

  ctx.lineJoin = "round";

  ctx.beginPath();

  ctx.moveTo(x1, y1);

  ctx.lineTo(x2, y2);

  ctx.stroke();
}
```

---

## Step 15 同步顏色

修改 emitDraw：

```js
function emitDraw(
  x1,
  y1,
  x2,
  y2
) {
  ws.emit("draw", {
    from: {
      x: x1,
      y: y1,
    },

    to: {
      x: x2,
      y: y2,
    },

    color,
  });
}
```

---

### 接收顏色

```js
ws.on("draw", (data) => {
  drawLine(
    data.from.x,
    data.from.y,
    data.to.x,
    data.to.y,
    data.color
  );
});
```

---

### Server 為什麼不用改？

因為：

# Server 不在乎資料內容

它只負責：

```text
收到 → 廣播
```

---

## Step 16 加入筆刷大小

建立狀態：

```js
let brushSize = 4;
```

---

### 建立 slider

```js
const sizeInput =
  document.createElement("input");

sizeInput.type = "range";

sizeInput.min = 1;

sizeInput.max = 30;

sizeInput.value = brushSize;

sizeInput.oninput = (e) => {
  brushSize =
    Number(e.target.value);
};

toolbar.appendChild(sizeInput);
```

---

### type="range"

HTML 內建：

# 滑桿元件

---

### 同步 brushSize

emit 時加入：

```js
size: brushSize
```

---

### drawLine 修改

```js
ctx.lineWidth = size;
```

---

## Step 17 Clear 功能

建立按鈕：

```js
const clearBtn =
  document.createElement("button");

clearBtn.innerText = "Clear";

toolbar.appendChild(clearBtn);
```

---

### 清空畫布

```js
clearBtn.onclick = () => {
  ctx.clearRect(
    0,
    0,
    canvas.width,
    canvas.height
  );

  ws.emit("clear");
};
```

---

### clearRect()

```js
ctx.clearRect(...)
```

把指定區域清空。

---

### Server 接收 clear

server.py：

```py
@sio.on("clear")
def clear(sid):
    sio.emit("clear")
```

---

### Client 接收 clear

```js
ws.on("clear", () => {
  ctx.clearRect(
    0,
    0,
    canvas.width,
    canvas.height
  );
});
```

---

## Step 18 手機觸控支援

目前只能滑鼠畫圖。

接下來加入手機觸控。

---

### touchstart

```js
canvas.addEventListener(
  "touchstart",
  (e) => {
    e.preventDefault();

    const touch =
      e.touches[0];

    drawing = true;

    current.x =
      touch.clientX;

    current.y =
      touch.clientY;
  }
);
```

---

### touchmove

```js
canvas.addEventListener(
  "touchmove",
  (e) => {
    e.preventDefault();

    if (!drawing) return;

    const touch =
      e.touches[0];

    const x = touch.clientX;
    const y = touch.clientY;

    drawLine(
      current.x,
      current.y,
      x,
      y,
      color
    );

    emitDraw(
      current.x,
      current.y,
      x,
      y
    );

    current.x = x;
    current.y = y;
  }
);
```

---

### touchend

```js
window.addEventListener(
  "touchend",
  () => {
    drawing = false;
  }
);
```

---

### e.preventDefault()

避免：

* 頁面捲動
* 縮放
* 手勢干擾

---

## 最重要的觀念

今天這個專案的核心不是畫圖，而是帶大家瞭解：

# 「即時同步系統」

這也是：

* 線上遊戲
* Discord
* Google Docs
* Line
* 多人白板

背後最重要，最核心的概念 0.0
