## Simple websocket server

Fork of [Cherry-Pie/websocket](https://github.com/Cherry-Pie/websocket) for Laravel 5 integration.

### Installation
```bash
composer require yaro/socket
```

Add to config/app.php:
```php
'providers' => array(
//...
    Yaro\Socket\ServiceProvider::class,
//...
),
'aliases' => array(
//...
    'Socket' => Yaro\Socket\Facade::class,
//...
),
```

### Usage
Sample command:

```php
namespace App\Console\Commands;

use Illuminate\Console\Command;

use Socket;

class ChatCommand extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'socket:chat {action?}';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Command description';

    /**
     * Create a new command instance.
     *
     * @return void
     */
    public function __construct()
    {
        parent::__construct();
    }

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $addr = '127.0.0.1';
        $port = '1234';

        $commands = array('start','stop','restart');
        if(!in_array($this->argument('action'), $commands)){
            $this->info("Need command! - start|stop|restart");
        }else{
            Socket::init($this->argument('action'), array(
                'class' => \App\Socket\Daemon::class,
                'pid' => '/tmp/websocket_chat.pid',
                'websocket' => 'tcp://'.$addr.':'.$port,
                //'localsocket' => 'tcp://127.0.0.1:8010',
                //'master' => 'tcp://127.0.0.1:8020',
                //'eventDriver' => 'event'
            ));
            $this->info('done');
        }
    }
}
```


Sample handler class in `app\Socket\Daemon.php`:
```php
namespace App\Socket;

use Yaro\Socket\Websocket\WebsocketDaemon;

class Daemon extends WebsocketDaemon
{
    protected function onOpen($connectionId, $info)
    {
        echo 'opened #'.$connectionId.PHP_EOL;
    }

    protected function onClose($connectionId) 
    {
        echo 'closed #'.$connectionId.PHP_EOL;
    }

    protected function onMessage($connectionId, $data, $type)
    {
        echo '#'.$connectionId.' -> '.$data.PHP_EOL;

        if (!strlen($data)) {
            return;
        }

        $message = 'user #'. $connectionId .' ('. $this->pid .'): '. strip_tags($data);

        foreach ($this->clients as $idClient => $client) {
            $this->sendToClient($idClient, $message);
        }
    }
}
```


And run your command (command from example):
```shell
nohup php artisan socket:chat start > /dev/null &
```


And your js on front will be like this:
```javascript
var ws = new WebSocket("ws://127.0.0.1:8000/");
ws.onopen = function() { 
    console.log('socket: open');
};
ws.onclose = function() { 
    console.log('socket: close');
};
ws.onmessage = function(evt) { 
    console.log(evt.data);
};
```
