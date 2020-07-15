[TOC]

# Lumen中接入消息通知

*说明： `lumen` 中默认是没有加载消息通知功能的，因业务需要进行消息通知，所以需要手动整合（以 `Laravel` 中的消息通知为原型）*

> 实现目标：
>
> 1. 实现消息通知
> 2. 可实现自定义消息通知驱动
> 3. 异步通知

## 默认支持消息通知驱动

`illuminate/notifications` 该拓展包已支持 邮件、短信、数据库以及 [Slack](https://slack.com/)等方式进行消息通知。

本教程将以 **邮件** 方式进行说明（其他驱动可做类似实现），并且自定义的消息通知方式为 企业微信群机器人。

## 引入拓展

1. 安装适配版本的依赖包 

   ```php
   composer require illuminate/notifications 5.8.*
   ```

2. 注册通知服务，在`bootstrap/app` 中注入`NotificationServiceProvider`

   ```php
   $app->register(\Illuminate\Notifications\NotificationServiceProvider::class);
   ```

3. 邮件服务相关配置，配置可参考 [邮件发送](https://learnku.com/docs/laravel/5.8/mail/3920) （ *注意：username和address要保证是同一个值*）

   ```php
   return [
     
       /*
       |--------------------------------------------------------------------------
       | Mail Driver
       |--------------------------------------------------------------------------
       |
       */
       'driver' => env('MAIL_DRIVER', 'smtp'),
     
       /*
       |--------------------------------------------------------------------------
       | SMTP Host Address
       |--------------------------------------------------------------------------
       |
       */
       'host' => env('MAIL_HOST', 'smtp.mailgun.org'),
     
       /*
       |--------------------------------------------------------------------------
       | SMTP Host Port
       |--------------------------------------------------------------------------
       |
       */
       'port' => env('MAIL_PORT', 587),
   
       /*
       |--------------------------------------------------------------------------
       | Global "From" Address
       |--------------------------------------------------------------------------
       |
       */
       'from' => [
           'address' => env('MAIL_FROM_ADDRESS', 'hello@example.com'),
           'name' => env('MAIL_FROM_NAME', 'Example'),
       ],
   
       /*
       |--------------------------------------------------------------------------
       | E-Mail Encryption Protocol
       |--------------------------------------------------------------------------
       |
       */
       'encryption' => env('MAIL_ENCRYPTION', 'tls'),
   
       /*
       |--------------------------------------------------------------------------
       | SMTP Server Account & Password
       |--------------------------------------------------------------------------
       |
       */
       'username' => env('MAIL_USERNAME'),
       'password' => env('MAIL_PASSWORD'),
   
   ]
   ```

4. 加载配置，注册邮件服务，在在`bootstrap/app` 中加载`mail.php`配置并注入`MailServiceProvider`

   ```php
   $app->configure('mail');
   
   $app->register(Illuminate\Mail\MailServiceProvider::class);
   
   $app->alias('mailer', \Illuminate\Contracts\Mail\Mailer::class);
   ```

5. 测试中发现若使用**邮件** 进行消息通知，则还需要手动引入`ramsey/uuid 依赖

   ```php
   composer require ramsey/uuid
   ```

## 创建通知

在 `laravel/lumen` 中，一条通知就是一个类。要使用消息通知时就创建一个类，该类必须继承自 `Illuminate\Notifications\Notification`，该类包含 `via` 方法以及一个或多个消息构建的方法，它们会针对指定的渠道把通知转换为对应的消息。

以下代码为邮件通知的模板内容：

```php
<?php

namespace App\Notifications;

use Illuminate\Notifications\Notification;;
use Illuminate\Notifications\Messages\MailMessage;

class InvoicePaid extends Notification
{
    /**
     * Create a new notification instance.
     *
     */
    public function __construct()
    {
        // TODO
    }

    /**
     * Get the notification's delivery channels.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function via($notifiable)
    {
        return ['mail'];
    }

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->line('The introduction to the notification.')
                    ->action('Notification Action', url('/'))
                    ->line('Thank you for using our application!');
    }
}

```

## 发送通知

通知可以通过两种方法发送：`Notifiable Trait` 的 `notify` 方法或 `Notification Facade`。

- 使用 `Notifiable Trait`

  此时 `AccessIp` 必须使用 `Notifiable Trait`

  ```php
  <?php
  
  namespace App\Models\OfficialAccount;
  
  use App\Models\Base;
  use Illuminate\Notifications\Notifiable;
  
  class AccessIp extends Base
  {
      use Notifiable;
  
      protected $table = 'access_ip';
  
      /**
       * 使用mail驱动时的必要属性
       * 
       * @var string 
       */
      protected $email = 'luoyanshou@vchangyi.com';
  
  }
  ```

  

  ```php
  <?php
  
  namespace App\Console\Commands;
  
  use App\Models\OfficialAccount\AccessIp;
  use App\Notifications\InvoicePaid;
  use Illuminate\Console\Command;
  
  class NoticeTest extends Command
  {
      protected $signature = 'feat:notice-test';
  
      protected $description = '';
  
      public function handle()
      {
          (new AccessIp())->notify(new InvoicePaid());
      }
  }
  
  ```

  

- 使用 ``Notification Facade`

  `AccessIp` 不必使用 `Notifiable Trait`

  ```php
  <?php
  
  namespace App\Console\Commands;
  
  use App\Models\OfficialAccount\AccessIp;
  use App\Notifications\InvoicePaid;
  use Illuminate\Console\Command;
  use Illuminate\Support\Facades\Notification;
  
  class NoticeTest extends Command
  {
      protected $signature = 'feat:notice-test';
  
      protected $description = '';
  
      public function handle()
      {
          Notification::send(new AccessIp(), new InvoicePaid());
      }
  }
  ```


## 自定义消息驱动

有时业务需要，默认的驱动方式可能不满足需求，这个时候我们可以实现自己的消息驱动。自定义消息通知驱动只需要实现一个 `send` 方法即可，以下为企业微信机器人消息通知简单实现：

```php
<?php

namespace App\Notifications\Channel;

use Illuminate\Notifications\Notification;

class WechatChannel
{
    public function send($notifiable, Notification $notification)
    {
        $content = [
            'msgtype' => 'text',
            'text' => [
                'content' => '消息通知测试'
            ]
        ];
        $url = 'https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=xxx';
        (new \GuzzleHttp\Client())->post($url, [
            'json' => $content,
        ]);

    }
}
```

## 使用自定义驱动

使用自定义消息驱动和使用默认的消息驱动一样，把消息驱动加载到通知类的 `via` 方法以及按需实现 `toXXX` 方法即可。

```php
/**
     * Get the notification's delivery channels.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function via($notifiable)
    {
        return ['mail', WechatChannel::class];
    }
```

```php
/**
     * 企业微信机器人消息通知
     * 
     * @param $notifiable
     * @return array
     */
    public function toWechat($notifiable)
    {
        return [
            // TODO
        ];
    }
```

## 异步通知处理

`illuminate/notifications` 该拓展包默认已支持异步通知功能，我们只用对已有的通知类做简单修改即可。

已有的通知类继承 `Illuminate\Notifications\Notification` 的同时再实现下 `Illuminate\Contracts\Queue\ShouldQueue` , 并且使用 `Queueable Trait`。代码样例如下：

```php
class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;
    
    ......
}
```

**以上只是把消息放入异步队列中，还需要启用一个work来实现消息的通知下发。**

## 应用场景

消息通知主要是告知用户应用发生了什么，常用于订单完成、评论回复、系统监控等场景下。

我们目前的开发，一些业务会用到定时任务，但是缺少对定时任务的监控策略，导致我们无法获知这些任务是否成功执行；在接下来的开发中，我们会将消息通知应用到定时任务的监控中。

有类似问题的业务，可以参考本教程完成相应的消息通知。



*参考链接： [消息通知](https://learnku.com/docs/laravel/5.8/notifications/3921)*