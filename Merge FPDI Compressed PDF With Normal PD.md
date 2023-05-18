Merge FPDI Compressed PDF With Normal PDF

There are two method to solve these problem which i know.

Method One :- Trail and Paid Method

https://www.setasign.com/products/fpdi-pdf-parser/details/

Register and login with setasign https://www.setasign.com/register/
Add below code into composer.json for php 8.1
    // "setasign/fpdf": "1.8.*",
    // "setasign/fpdi": "^2.3",
    // "setasign/fpdi_pdf-parser_eval_ioncube_php8.1": "2.0.7",
    
and run in  terminal  `composer update`
it ask your email or username and password. This package provide for free trail for some days and paid version.
you don't need to any thing when you use this package.

Method Two :- Free Method

This is the free method to make merge PDF, the objective to convert PDF to image. With that image we re-convert into pdf

For doing this you need follow 
1. GPL Ghostscript exe file. (https://ghostscript.com/releases/gsdnld.html)
2. ImageMagick exe file. (https://imagemagick.org/script/download.php)
3. ioncube_loader_win_8.1.dll (https://www.ioncube.com/loaders.php)
4. php_imagick.dll (https://mlocati.github.io/articles/php-windows-imagick.html)

Firstly, I have PHP 7.4 i have run exe file and upload .dll file in D:\xampp\php\ext and add extension in php.ini file
but i won't work on PHP 7.4 version so the conclusion is you have to make your system compatible with exe and .dll file.
so, I switch to PHP 8.1 I provide all the software .exe and .dll file for PHP 8.1 version and also the link so you can download the as your PHP version.

I start install for PHP 8.1 version whatever php version you have after getting exe and dll file as your version required you need to follow below steps
Install Ghostscript and ImageMagick exe file into the system.
Upload ioncube_loader and php_imagick dll file into D:\xampp\php\ext
now open php.ini file and add `zend_extension = "D:\xampp\php\ext\ioncube_loader_win_8.1.dll"` and `extension=php_imagick.dll` in php.ini file.
restart your server.

You can check imagick is install or not in http://localhost/dashboard/phpinfo.php


You need to implement code in your cotroller.

    use Spatie\PdfToImage\Pdf as SpatiePDF;

    public static function PDFConvertToImg($fileName , $arrayData)
    {
        $compressedPdfPath = storage_path('app/public/chunks/' . $fileName);
        $storePdfPath = storage_path('app/public/chunks');
        $pdf = new SpatiePDF($compressedPdfPath);
        $saveImages = $pdf->saveAllPagesAsImages($storePdfPath);
        if(count($saveImages) > 0){
            $i = 1;
            $pdfMerger = new \Jurosh\PDFMerge\PDFMerger;
            foreach($saveImages as $data){
                $savePDF = storage_path('app/public/chunks/'.$i.'.pdf');

                $path = $data;
                $ext = pathinfo($path, PATHINFO_EXTENSION);
                $pdffilename = $i.'.'.$ext;
                $aws_file_path = env('S3_BUCKET_FOLDER_NAME') .'/'. $pdffilename;
                Storage::disk('s3')->put($aws_file_path,fopen($path,'r'),'public');
                $pdfUrl = Storage::disk('s3')->url($aws_file_path);
                $url = $pdfUrl;
                $pdf = PDF::loadView('emailTemplate.bolpdf', ['a3' => $url ]);
                $pdf->save($savePDF);

                $pdfMerger->addPDF($savePDF, 'all');
                unlink($path);
                $i++;
            }
            $mergepdffileURL = $pdfMerger->merge('file', storage_path('app/public/chunks/mergefile.pdf') );
            $mergeFileURL = storage_path('app/public/chunks/mergefile.pdf');
            $mergepdffilename = rand().'_mergefile.pdf';
            $aws_file_path_for_merge = env('S3_BUCKET_FOLDER_NAME') .'/'. $mergepdffilename;
            Storage::disk('s3')->put($aws_file_path_for_merge,fopen($mergeFileURL,'r'),'public');
            $mergePdfUrl = Storage::disk('s3')->url($aws_file_path_for_merge);

            $j = 1;
            foreach($saveImages as $data){
                $deletePDF = storage_path('app/public/chunks/'.$j.'.pdf');
                unlink($deletePDF);
                $j++;
            }
            unlink($mergeFileURL);
            unlink($compressedPdfPath);
        }
    }

So, this will work for me. Please let me know if you have any problem in execution.
