1. В поиске найти пользователя @BotFather
2. Отправить сообщение боту — /newbot
3. Указать имя бота
4. Продублировать название бота, но только суффиксом _bot
5. @BotFather пришлёт сообщение с токеном, который вам нужно сохранить
6. Создать чат, добавить в него бота
7. Перейти по ссылке, где вместо символов X нужно подставить токен, страницу не закрывать https://api.telegram.org/botXXXXXXXXXXXXXXXXXX/getUpdates
8. Отправить команду /join в чат для активации бота
9. Обновить страницу п.7, чтобы сделать повторный запрос
10. Скопировать id со знаком минус
11. cron:
- local: php artisan schedule:work
- server: * * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1

.env
````
TG_TOKEN=
TG_CHAT_ID=
````

app/Services/TelegramService.php

```
namespace App\Services;

use Illuminate\Support\Facades\Http;

class TelegramService
{
    public function sendMessage(string $name, string $url): void
    {
        $token = config('services.env_var.tg_token');

        Http::get('https://api.telegram.org/bot' . $token . '/sendMessage',
            [
                "chat_id" => config('services.env_var.tg_chat_id'),
                "parse_mode" => 'HTML',
                "text" => ''
            ]
        );
    }
}
```

app/Services/ParsingService.php

```
namespace App\Services;

use DOMDocument;
use DOMXPath;

class ParsingService
{
    public function parse(TelegramService $telegramService): void
    {
        $urls = []; // <=
        
        foreach ($urls as $url) {
            $html = file_get_contents($url);
            $dom = new DOMDocument();
            @$dom->loadHTML($html);
            $xpath = new DOMXPath($dom);
            //some parsing 
            //$xpath->query('//h1')[0]->nodeValue
            //$xpath->query('//td[@class="some-class"]')[0];
            $telegramService->sendMessage();
        }
    }
```

app/Console/Kernel.php

```
protected function schedule(Schedule $schedule): void
    {
        $schedule->call(function () {
            $telegramService = new TelegramService();
            $parsingService = new ParsingService();
            $parsingService->parse($telegramService);
        })->everyMinute(); // <=
    }
```


