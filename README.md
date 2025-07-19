// 1. وارد کردن کتابخانه‌های مورد نیاز
const express = require('express'); // برای ساخت سرور وب
const http = require('http');     // ماژول داخلی Node.js برای سرور HTTP
const WebSocket = require('ws');  // برای ساخت سرور WebSocket

// 2. راه‌اندازی سرور Express (سرور وب)
const app = express();
const server = http.createServer(app); // ساخت سرور HTTP با Express

// 3. راه‌اندازی سرور WebSocket
const wss = new WebSocket.Server({ server }); // اتصال WebSocket به سرور HTTP موجود

// 4. مدیریت اتصال‌های WebSocket
// وقتی یک کاربر جدید به سرور WebSocket متصل میشه:
wss.on('connection', ws => {
    console.log('یک کاربر جدید متصل شد!'); // این پیام رو در ترمینال کُداسپِیسز می‌بینید

    // وقتی سرور از کاربر پیامی دریافت می‌کنه:
    ws.on('message', message => {
        const messageString = message.toString(); // پیام رو به رشته تبدیل می‌کنیم
        console.log(`پیام دریافت شد: ${messageString}`); // پیام رو در ترمینال نمایش میدیم

        // پیام رو برای همه کاربران متصل دیگه (به جز خودش) ارسال می‌کنیم
        wss.clients.forEach(client => {
            if (client !== ws && client.readyState === WebSocket.OPEN) {
                client.send(messageString); // پیام رو به کلاینت‌های دیگه ارسال می‌کنه
            }
        });
    });

    // وقتی یک کاربر قطع اتصال می‌کنه:
    ws.on('close', () => {
        console.log('یک کاربر قطع اتصال کرد.');
    });

    // پیامی برای کاربر جدید ارسال می‌کنیم
    ws.send('به پیام‌رسان ساده ما خوش آمدید!');
});

// 5. تنظیم یک مسیر برای صفحه اصلی وب (اختیاری، اما برای تست خوبه)
app.get('/', (req, res) => {
    res.send(`
        <!DOCTYPE html>
        <html>
        <head>
            <title>پیام‌رسان ساده</title>
            <style>
                body { font-family: sans-serif; margin: 20px; background-color: #f0f2f5; }
                #messages { border: 1px solid #ccc; padding: 10px; height: 300px; overflow-y: scroll; background-color: #fff; margin-bottom: 10px; }
                #messageInput { width: calc(100% - 80px); padding: 8px; border: 1px solid #ccc; border-radius: 4px; }
                button { padding: 8px 15px; background-color: #007bff; color: white; border: none; border-radius: 4px; cursor: pointer; }
                button:hover { background-color: #0056b3; }
            </style>
        </head>
        <body>
            <h1>پیام‌رسان ساده</h1>
            <div id="messages"></div>
            <input type="text" id="messageInput" placeholder="پیام خود را بنویسید...">
            <button onclick="sendMessage()">ارسال</button>

            <script>
                // اتصال به سرور WebSocket
                const ws = new WebSocket('ws://' + window.location.host); // آدرس سرور WebSocket

                // وقتی پیامی از سرور دریافت میشه
                ws.onmessage = event => {
                    const messagesDiv = document.getElementById('messages');
                    const p = document.createElement('p');
                    p.textContent = event.data;
                    messagesDiv.appendChild(p);
                    messagesDiv.scrollTop = messagesDiv.scrollHeight; // اسکرول به پایین
                };

                // تابع ارسال پیام
                function sendMessage() {
                    const input = document.getElementById('messageInput');
                    const message = input.value;
                    if (message.trim() !== '') {
                        ws.send(message); // ارسال پیام به سرور
                        input.value = ''; // پاک کردن ورودی
                    }
                }

                // ارسال پیام با فشردن Enter
                document.getElementById('messageInput').addEventListener('keypress', function(e) {
                    if (e.key === 'Enter') {
                        sendMessage();
                    }
                });
            </script>
        </body>
        </html>
    `);
});

// 6. گوش دادن سرور به یک پورت خاص
const PORT = process.env.PORT || 3000; // از پورت 3000 استفاده می‌کنیم یا پورتی که کُداسپِیسز میده
server.listen(PORT, () => {
    console.log(`سرور پیام‌رسان در حال اجرا روی پورت ${PORT}`);
    console.log('می‌توانید پیش‌نمایش را در تب "Ports" ببینید.');
});
