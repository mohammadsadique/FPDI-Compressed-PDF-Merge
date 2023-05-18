# Merge FPDI Compressed PDF With Normal PDF

There are two method to solve these problem which i know.
The first method involves using a third-party package called FPDI PDF Parser, which requires registration and login with the Setasign website. Once registered, the package can be installed using Composer by adding the required dependencies to the composer.json file. This method provides both a free trial and a paid version, and it is straightforward to use.

The second method is a free solution that involves converting the PDF into an image using Ghostscript and ImageMagick. Once converted, the images are re-converted into a PDF using Spatie PDFToImage. The necessary files for Ghostscript, ImageMagick, ioncube_loader_win_8.1.dll, and php_imagick.dll need to be downloaded and installed on the system. The process involves modifying the php.ini file and checking if Imagick is installed properly. The code for implementing the solution involves using the Spatie\PdfToImage\Pdf class to save all pages of the PDF as images and then converting them back into a PDF using the Jurosh\PDFMerge\PDFMerger class.

Overall, the code you provided appears to be a feasible solution for merging a compressed PDF with a normal PDF, but it may require some technical expertise and installation of additional software and packages.

## Method One: Trial and Paid Method

To merge a compressed PDF with a normal PDF, you can use a third-party package called FPDI PDF Parser, which provides both a free trial and a paid version. Here's how to get started:

1. Register and log in to Setasign website: https://www.setasign.com/register/
2. Add the following code to your composer.json file for PHP 8.1



In your composer.json file, add the following dependencies:

```
"require": {
    "setasign/fpdf": "1.8.*",
    "setasign/fpdi": "^2.3",
    "setasign/fpdi_pdf-parser_eval_ioncube_php8.1": "2.0.7"
}
```



## Method Two :- Free Method


This is a free method for merging PDF files, which involves converting PDFs to images and then re-converting them back into PDF format. Follow the steps below to proceed:

1. Download the GPL Ghostscript executable file from [here](https://ghostscript.com/releases/gsdnld.html).
2. Download the ImageMagick executable file from [here](https://imagemagick.org/script/download.php).
3. Download the `ioncube_loader_win_8.1.dll` file from [here](https://www.ioncube.com/loaders.php).
4. Download the `php_imagick.dll` file from [here](https://mlocati.github.io/articles/php-windows-imagick.html).

Make sure you have these necessary files available for the process to work correctly.

## Setting Up Environment for PHP 8.1

If you're currently using PHP 7.4 and experiencing compatibility issues, follow the steps below to switch to PHP 8.1 and configure the necessary components:

1. Download the required PHP 8.1 executable and DLL files based on your PHP version from the official website.
2. Install the Ghostscript and ImageMagick executable files on your system.
3. Upload the `ioncube_loader` and `php_imagick` DLL files to the `D:\xampp\php\ext` directory.
4. Open the `php.ini` file.
5. Add the following lines to the `php.ini` file:
6. Save the changes to the `php.ini` file.
7. Restart your server to apply the configuration changes.

You can verify whether Imagick is installed correctly by accessing [http://localhost/dashboard/phpinfo.php](http://localhost/dashboard/phpinfo.php) and checking for the Imagick module.




You need to implement code in your cotroller.

```
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

```

So, this will work for me. Please let me know if you have any problem in execution.







## Contact

Name - [Md Sadique](mailto:sadiquedeveloper@gmail.com)




