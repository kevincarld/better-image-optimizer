# better-image-optimizer-next


[![npm](https://img.shields.io/npm/v/better-image-optimizer-next)](https://www.npmjs.com/package/better-image-optimizer-next)



>  **Warning**

> Only use with Next 13's new next/image component.



Use [Next.js advanced **\<Image/>** component](https://nextjs.org/docs/basic-features/image-optimization) with the static export functionality. Optimizes all static images in an additional step after the Next.js static export.
Based from library [next-image-export-optimizer](https://github.com/Niels-IO/next-image-export-optimizer#readme) - credits to Niels-IO


- Reduces the image size and page load times drastically through responsive images

- Fast image transformation using [sharp.js](https://www.npmjs.com/package/sharp) (also used by the Next.js server in production)

- Conversion of JPEG and PNG files to the modern WEBP format by default

- Serve the exported React bundle only via a CDN. No server required

- Automatic generation of tiny, blurry placeholder images

- Minimal configuration necessary


This library makes a few assumptions:



- All images that should be optimized are stored inside the public folder like public/images (except for the statically imported images)

- Currently only local images are supported for optimization



## Installation



```

npm install better-image-optimizer-next
# Or

yarn add better-image-optimizer-next

pnpm install better-image-optimizer-next

```



Configure the library in your **Next.js** configuration file:



```javascript

// next.config.js

module.exports = {

images: {

loader:  "custom",

imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],

deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],

},

env: {

nextImageExportOptimizer_imageFolderPath:  "public/images",

nextImageExportOptimizer_exportFolderPath:  "out",

nextImageExportOptimizer_quality:  75,

nextImageExportOptimizer_storePicturesInWEBP:  true,



// If you do not want to use blurry placeholder images, then you can set

// nextImageExportOptimizer_generateAndUseBlurImages to false and pass

// `placeholder="empty"` to all <ExportedImage> components.

//

// If nextImageExportOptimizer_generateAndUseBlurImages is false and you

// forget to set `placeholder="empty"`, you'll see 404 errors for the missing

// placeholder images in the console.

nextImageExportOptimizer_generateAndUseBlurImages:  true,

},

};

```



1. Add the above configuration to your **next.config.js**

2. Specify the folder where all the images are stored. Defaults to **public/images**

3. Change the export command in `package.json`



```diff

{

- "export": "next build && next export",

+ "export": "next build && next export && better-image-optimizer-next"

}

```



If your nextjs project is not at the root directory where you are running the commands, for example if you are using a monorepo, you can specify the location of the next.config.js as an argument to the script:



```json

"export": "next build && next export && better-image-optimizer-next path/to/my/next.config.js"

```



4. Change the **\<Image />** component to the **\<ExportedImage />** component of this library.



Example:



```javascript

// Old

import  Image  from  "next/image";



<Image

src="images/VERY_LARGE_IMAGE.jpg"

alt="Large Image"

width={500}

height={500}

/>;



// New

import  BetterImage  from  "next-image-export-optimizer";



<BetterImage

src="images/VERY_LARGE_IMAGE.jpg"

alt="Large Image"

width={500}

height={500}

useWebp={process.env.nextImageExportOptimizer_storePicturesInWEBP}

/>;



// Or with static import

import  BetterImage  from  "next-image-export-optimizer";

import  testPictureStatic  from  "PATH_TO_IMAGE/test_static.jpg";



<BetterImage

src={testPictureStatic}

alt="Static Image"

layout="responsive"

useWebp={process.env.nextImageExportOptimizer_storePicturesInWEBP}

/>;

```



5. In the development mode, either the original image will be served as a fallback when the optimized images are not yet generated or the optimized image once the image transformation was executed for the specific image. The optimized images are created at build time only. In the exported, static React app, the responsive images are available as srcset and dynamically loaded by the browser.



6. You can output the original, unoptimized images using the `unoptimized` prop.

Example:



```javascript

<ExportedImage

src={testPictureStatic}

alt="Orginal, unoptimized image"

unoptimized={true}

/>

```



7. Overriding presets:



**Placeholder images:**

If you do not want the automatic generation of tiny, blurry placeholder images, set the `nextImageExportOptimizer_generateAndUseBlurImages` environment variable to `false` and set the `placeholder` prop from the **\<ExportedImage />** component to `empty`.



**Usage of the WEBP format:**

If you do not want to use the WEBP format, set the `nextImageExportOptimizer_storePicturesInWEBP` environment variable to `false` and set the `useWebp` prop from the **\<ExportedImage />** component to `false`.


## How it works



The **\<BetterImage />** component of this library wraps around the **\<Image />** component of Next.js. Using the custom loader feature, it generates a [srcset](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images) for different resolutions of the original image. The browser can then load the correct size based on the current viewport size.



In the development mode, the **\<BetterImage />** component falls back to the original image.



All images in the specified folder, as well as all statically imported images will be optimized and reduced versions will be created based on the requested widths.



The image transformation operation is optimized as it uses hashes to determine whether an image has already been optimized or not.