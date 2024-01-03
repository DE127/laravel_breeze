## CKEditor 5

CKEditor 5 là một trình soạn thảo văn bản phong phú JavaScript với nhiều tính năng và khả năng tùy chỉnh.

CKEditor 5 có kiến trúc MVC hiện đại, mô hình dữ liệu tùy chỉnh và DOM ảo, mang lại hiệu suất và khả năng mở rộng vượt trội.

Tích hợp gốc với Angular, React và Vue.js có sẵn để thuận tiện cho bạn cũng như tương thích với Electron và các thiết bị di động (Android, iOS).

## Laravel Vite

Laravel Vite là sự kết hợp giữa Laravel PHP framework và Vite, một công cụ xây dựng hiện đại để phát triển giao diện người dùng. Nó cho phép bạn sử dụng Vite làm hệ thống xây dựng giao diện người dùng cho các ứng dụng Laravel của mình.

Vite được biết đến với máy chủ phát triển nhanh và quy trình xây dựng hiệu quả.

Nó tận dụng ES modules gốc, với các tính năng tích hợp mạnh mẽ và khả năng thay thế Hot Module Replacement (HMR) vô cùng nhanh chóng.

## Build CKEditor 5 bằng Laravel Vite

Trong bài viết này, tôi sẽ hướng dẫn các bạn cách build CKEditor 5 sử dụng Laravel Vite trong một dự án Laravel (Phiên bản được áp dụng trong bài viết là Laravel 10).

Trước khi bắt đầu, hãy đảm bảo rằng bạn đã cài đặt Node.js phiên bản 14.18 hoặc cao hơn, hoặc phiên bản 16 trở lên, vì đây là yêu cầu để sử dụng Vite.

Đầu tiên, chúng ta sẽ khởi tạo dự án Laravel mới. Bạn có thể làm điều này bằng cách sử dụng lệnh sau:

```php
composer create-project laravel/laravel ckeditor5
```

Tiếp theo, chúng ta sẽ cài đặt các packages cơ bản của CKEditor 5 gồm có Editor base, Editor plugins và Editor theme.

Dưới đây là một danh sách mẫu các plugin cơ bản:

```php
npm install --save @ckeditor/ckeditor5-theme-lark \
  @ckeditor/ckeditor5-autoformat \
  @ckeditor/ckeditor5-basic-styles \
  @ckeditor/ckeditor5-block-quote \
  @ckeditor/ckeditor5-editor-classic \
  @ckeditor/ckeditor5-essentials \
  @ckeditor/ckeditor5-heading \
  @ckeditor/ckeditor5-link \
  @ckeditor/ckeditor5-list \
  @ckeditor/ckeditor5-paragraph
```

Khi CKEditor 5 của bạn đã có đầy đủ các plugin cần thiết, chúng ta có thể tiến hành tích hợp với Vite.

Có một plugin chính thức hỗ trợ chúng ta trong mục đích này. Plugin này xử lý tải SVG icons và Styles từ các package và theme package. Bạn có thể cài đặt nó bằng lệnh dưới đây:

```php
npm install --save @ckeditor/vite-plugin-ckeditor5
```

Sau khi, cài đặt plugin trên thì nó vẫn chưa thể hoạt động được, chúng ta phải thêm nó vào cấu hình của Vite bằng cách chỉnh sửa tập tin `vite.config.js` với nội dung như sau:

```javascript
import { createRequire } from 'node:module';
const require = createRequire( import.meta.url );
 
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import ckeditor5 from '@ckeditor/vite-plugin-ckeditor5';
 
export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js', 'resources/css/ckeditor.css', 'resources/js/ckeditor.js'],
            refresh: true,
        }),
        ckeditor5({
            theme: require.resolve( '@ckeditor/ckeditor5-theme-lark' )
        }),
    ],
});
```

Tiếp theo, chúng ta sẽ tạo tập tin `ckeditor.js` trong thư mục `src/resources/js` với nội dung như sau:

```javascript
import { ClassicEditor as ClassicEditorBase } from '@ckeditor/ckeditor5-editor-classic';
import { Essentials } from '@ckeditor/ckeditor5-essentials';
import { Autoformat } from '@ckeditor/ckeditor5-autoformat';
import { Bold, Italic } from '@ckeditor/ckeditor5-basic-styles';
import { BlockQuote } from '@ckeditor/ckeditor5-block-quote';
import { Heading } from '@ckeditor/ckeditor5-heading';
import { Link } from '@ckeditor/ckeditor5-link';
import { List } from '@ckeditor/ckeditor5-list';
import { Paragraph } from '@ckeditor/ckeditor5-paragraph';

export default class ClassicEditor extends ClassicEditorBase {}

ClassicEditor.builtinPlugins = [
    Essentials,
    Autoformat,
    Bold,
    Italic,
    BlockQuote,
    Heading,
    Link,
    List,
    Paragraph,
];

ClassicEditor.defaultConfig = {
    toolbar: {
        items: [
            'heading',
            '|',
            'bold',
            'italic',
            'link',
            'bulletedList',
            'numberedList',
            'blockQuote',
            'undo',
            'redo',
        ]
    },
    language: 'en'
};


ClassicEditor
    // Note that you do not have to specify the plugin and toolbar configuration — using defaults from the build.
    .create( document.querySelector( '#editor' ))
    .then( editor => {
        console.log( 'Editor was initialized', editor );
    } )
    .catch( error => {
        console.error( error.stack );
    } );
```

Vì chiều cao mặc định của Editor có phần hạn chế, chúng ta sẽ điều chỉnh nó thông qua CSS. Hãy tạo tập tin `ckeditor.css` nằm trong thư mục `src/resources/css` với nội dung như sau:

```css
.ck-editor__editable_inline {
    min-height: 400px;
}
```

Cấu hình build CKEditor 5 bằng Laravel Vite đã sẵn sàng, nhưng hiện tại chúng ta không có giao diện để có thể hiển thị Editor. Do đó, chúng ta sẽ chỉnh sửa tập tin `welcome.blade.php` trong thư mục `src/resources/views` với nội dung như sau:

```php
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="description" content="ManhDan Blogs">
    <meta name="author" content="ManhDan Blogs">
    <meta name="generator" content="ManhDan Blogs 0.84.0">
    <title>CKEditor 5 – Classic Editor</title>
    <link rel="icon" href="https://manhdandev.com/web/img/favicon.webp" type="image/x-icon"/>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
    @vite(['resources/css/ckeditor.css', 'resources/js/ckeditor.js'])
</head>
<body>
    <div class="col-lg-8 mx-auto p-3 py-md-5">
    <header class="d-flex align-items-center pb-3 mb-3 border-bottom">
        <a href="https://manhdandev.com" class="d-flex align-items-center text-dark text-decoration-none" target="_blank">
            <img src="https://manhdandev.com/web/img/logo.webp" width="100px" height="100px">
        </a>
    </header>
    <main>
        <div id="editor">
            <p>This is some sample content.</p>
        </div>
    </main>
    <footer class="pt-5 my-5 text-muted border-top">
        ©Copyright ©2023 All rights reserved | This template is made with
        <i class="fa fa-heart-o"></i> by <a href="https://blog.dane.dev/" rel="noopener" target="_blank">ManhDanBlogs</a>
    </footer>
    </div>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

## Kết quả của công việc bạn đã làm đang chờ bạn khám phá!

Sau khi đã hoàn thành các bước trên, giờ là lúc để chúng ta cùng nhau khám phá thành quả công sức của mình.

Hãy thực thi lệnh sau để tiến hành build CKEditor 5 sử dụng Laravel Vite:

```php
npm run build
```

Cuối cùng, chúng ta hãy mở trình duyệt lên và truy cập vào địa chỉ  `http://127.0.0.1` để chiêm ngưỡng kết quả do chính bản thân chúng ta tạo ra 🤤🤤🤤🏆🍨🍨🍨.
