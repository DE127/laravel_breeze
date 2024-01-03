## Cài đặt Google Drive trong Laravel

Google đã cung cấp API cho chúng ta tương tác:

[https://developers.google.com/drive/api/v3/manage-uploads](https://developers.google.com/drive/api/v3/manage-uploads)

Nhưng để chúng ta có thể cài đặt nhanh chóng và dễ dàng sử dụng, mình sẽ dùng thư viện sau để hỗ trợ:

[https://github.com/masbug/flysystem-google-drive-ext](https://github.com/masbug/flysystem-google-drive-ext)

Nếu dự án của bạn sử dụng Flysystem V2/V3 hoặc Laravel >= 9.x.x thì bạn hãy sử dụng lệnh sau:

```php
composer require masbug/flysystem-google-drive-ext
```

Nếu dự án của bạn sử dụng Flysystem V1 hoặc Laravel <= 8.x.x thì bạn hãy sử dụng lệnh sau:

```php
composer require masbug/flysystem-google-drive-ext:"^1.0.0"
```

Sau khi, cài đặt thư viện hoàn thành, chúng ta sẽ dùng lệnh sau để tạo Service Providers cho Google Drive:

```php
php artisan make:provider GoogleServiceProvider
```

Tiếp theo, mở GoogleServiceProvider.php nằm trong thư mục app/Providers và chỉnh sửa như sau:

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Storage;

class GoogleServiceProvider extends ServiceProvider
{
    /**
     * Register services.
     *
     * @return void
     */
    public function register()
    {
        //
    }

    /**
     * Bootstrap services.
     *
     * @return void
     */
    public function boot()
    {
        try {
            Storage::extend('google', function($app, $config) {
                $options = [];
                if (!empty($config['teamDriveId'] ?? null)) {
                    $options['teamDriveId'] = $config['teamDriveId'];
                }
                $client = new \Google\Client();

                $client->setClientId($config['clientId']);
                $client->setClientSecret($config['clientSecret']);
                $client->refreshToken($config['refreshToken']);

                $service = new \Google\Service\Drive($client);
                $adapter = new \Masbug\Flysystem\GoogleDriveAdapter($service, $config['folder'] ?? '/', $options);
                $driver  = new \League\Flysystem\Filesystem($adapter);

                return new \Illuminate\Filesystem\FilesystemAdapter($driver, $adapter);
            });
        } catch(\Exception $e) {
            // your exception handling logic
        }
    }
}
```

Tiếp tục, mở file config/app.php và đăng kí GoogleServiceProvider như sau:

```php
....
'providers' => [
    ....
    App\Providers\GoogleServiceProvider::class,
],
....
```

Cuối cùng, chúng ta sẽ đăng kí thông tin Google Drive vào config/filesystems.php như sau:

```php
...
'disks' => [
    ...
    'google' => [
        'driver' => 'google',
        'clientId' => env('GOOGLE_DRIVE_CLIENT_ID'),
        'clientSecret' => env('GOOGLE_DRIVE_CLIENT_SECRET'),
        'refreshToken' => env('GOOGLE_DRIVE_REFRESH_TOKEN'),
        'folder' => env('GOOGLE_DRIVE_FOLDER'),
        //'teamDriveId' => env('GOOGLE_DRIVE_TEAM_DRIVE_ID'),
    ],
    ...
],
...
```

## Google Drive API keys

Để chúng ta có thể lưu trữ được trên Google Drive chúng ta cần có các thông tin như Client ID, Client Secret và Refresh Token. Các bạn có thể tham khảo các hướng dẫn bên dưới:

+ [Lấy thông tin Client ID và Client Secret](https://github.com/manh-dan/document/blob/main/google-drive-as-filesystem-in-laravel.md#l%E1%BA%A5y-th%C3%B4ng-tin-client-id-v%C3%A0-client-secret)
+ [Lấy thông tin Refresh Token](https://github.com/manh-dan/document/blob/main/google-drive-as-filesystem-in-laravel.md#l%E1%BA%A5y-th%C3%B4ng-tin-refresh-token)

Sau khi có được các thông tin cần thiết, bạn hãy thêm các thông tin đó vào .env như sau:

```php
FILESYSTEM_CLOUD=google
GOOGLE_DRIVE_CLIENT_ID=xxx
GOOGLE_DRIVE_CLIENT_SECRET=xxx
GOOGLE_DRIVE_REFRESH_TOKEN=xxx
GOOGLE_DRIVE_FOLDER=
```

※Nếu GOOGLE_DRIVE_FOLDER không có giá trị thì mặc định sẽ được upload ở thư mục gốc.

※Nếu GOOGLE_DRIVE_FOLDER là một tên thư mục không tồn tại thì khi upload sẽ tự động tạo ra thư mục đó và upload dữ liệu vào thư mục này.

## Trải nghiệm Upload File lên Google Drive trong Laravel

Đầu tiên, bạn mở routes/web.php và thêm route bên dưới:

```php
Route::get('upload-file', function() {
    \Storage::disk('google')->put('google-drive.txt', 'Google Drive As Filesystem In Laravel (ManhDanBlogs)');
    dd('Đã upload file lên google drive thành công!');
});
```

Tiếp theo, các bạn hãy truy cập vào URL: [http://127.0.0.1:8000/upload-file](http://127.0.0.1:8000/upload-file)

![](https://manhdandev.com/images/large/google-drive-as-filesystem-in-laravel-2-1646224986.webp)

Như vậy, chúng ta đã upload thành công lên Google Drive để chắc chắn chúng ta hãy truy cập vào Google Drive để kiểm tra:

![](https://manhdandev.com/images/large/google-drive-as-filesystem-in-laravel-1-1646224994.webp)
