---
title: Adobe Photoshop API features
description: Learn about the available features in Photoshop API.
keywords:
  - Adobe Photoshop API features
  - Photoshop API features
contributors:
  - https://github.com/archyposada
   - https://github.com/khound
hideBreadcrumbNav: true
---

# Supported Features

With Adobe Photoshop API, which is now available through Adobe Firefly Services, you can perform numerous rich image-enhancements at scale.

## Photoshop Actions

*Actions* are a series an image adjustment that you can record in the Photoshop app and then you can later apply to one or more images.

Typically, you apply actions on a single file or a batch of files and you record actions from Photoshop app menu commands, panel options, tool actions, and more.

![alt image](./frogs_adjusted.png?raw=true "Original Image")

For example, you can create an action file that changes the hue and saturation of an image, applies a slight blur effect to the image, and then saves the file in the desired format. In the Photoshop app, you create a sequence of actions and then save these image adjustments in a single file, known as an ATN file. You can later call the Photoshop API to run one or more actions from your ATN file and apply these actions to multiple images.

For background information on Photoshop Actions, see the <a href="https://helpx.adobe.com/photoshop/using/actions-actions-panel.html" target="_blank">Adobe Help Center.</a>

### Supported Input and Output formats

We support the following file formats for Photoshop API with Photoshop Actions:

* PSD
* JPEG
* PNG
* TIFF

### Best Practices and Limitations

* We support all Photoshop app dialogs, however we do not support interactions with operating system dialogs. The later means you cannot use Photoshop API to programmatically open a system-level print settings dialog. 
* We recommended creating Actions that do not require user intervention such as confirming a selection or providing file paths.
* Make sure to test your actions on Photoshop, with several different input/images. If it has any errors in Photoshop, it won't run successfully on our servers either.

The following are known limitations which Photoshop API does not support:

* Photoshop 3D, and Video and animation features features.
* Custom presets, for example color swatches and brushes.
* This endpoint does not currently support multiple file inputs.

You can choose to playback all of the tasks recorded in an Action or you can selectively choose a particular task from an Actions file and exclude the rest.

### Examples

In this example we applied a custom Action called Posterize. This ATN file had multiple recorded Photoshop tasks including Select Subject, Remove Background, Posterize, and Export as PNG.

![alt image](./spaniels_before_after.png?raw=true "Original Image")

```shell
curl -X POST \
  https://image.adobe.io/pie/psdService/photoshopActions \
  -H "Authorization: Bearer $token"  \
  -H "x-api-key: $apiKey" \
  -H "Content-Type: application/json" \
  -d '{
  "inputs": [
    {
      "href": "https://as2.ftcdn.net/jpg/02/49/48/49/500_F_249484911_JifPIzjUqzkRhcdMkF9GnsUI9zaqdAsn.jpg",
      "storage": "external"
    }
  ],
  "options": {
    "actions": [
      {
        "href": "https://raw.githubusercontent.com/johnleetran/ps-actions-samples/master/actions/Posterize.atn",
        "storage": "external"
      }
    ]
  },
  "outputs": [
    {
      "storage": "<storage>",
      "type": "image/jpeg",
      "href": "https://some-presigned-url/output.jpeg"
    }
  ]
}'
```

For another example, see [Execute Individual Photoshop Action](../code-sample/index.md#photoshop-actions-play-a-specific-action)

## ActionJSON Endpoint

Similar to the Photoshop Actions endpoint, this endpoint also helps you to apply the contents of ATN file to an image programmatically. However, there are a few key differences which give you added flexibility.

* You can modify the payload, such as adding an action.
* You don’t need to upload and store your ATN file at Firefly Services as you do with the Photoshop Actions endpoint.

![alt image](./spanielsBW.png?raw=true "Original Image")

In this example take the input image and ATN file from the previous example and in our script, we modify the Action to execute all of the same tasks with an additional step.

```shell
curl -X POST \
  https://image.adobe.io/pie/psdService/actionJSON \
  -H "Authorization: Bearer $token"  \
  -H "x-api-key: $apiKey" \
  -H "Content-Type: application/json" \
  -d '{
  "inputs": [{
  "href": "<SIGNED_GET_URL>",
  "storage": "<storage>"
  }],
  "options": {
    "actionJSON": [{
        "_obj": "imageSize",
        "constrainProportions": true,
        "interfaceIconFrameDimmed": {
          "_enum": "interpolationType",
          "_value": "automaticInterpolation"
        },
        "scaleStyles": true
      }, {
        "_obj": "imageSize",
        "constrainProportions": true,
        "interfaceIconFrameDimmed": {
          "_enum": "interpolationType",
          "_value": "automaticInterpolation"
        },
        "resolution": {
          "_unit": "densityUnit",
          "_value": 72.0
        },
        "scaleStyles": true
      },
      {
        "_obj": "make",
        "_target": [{
          "_ref": "adjustmentLayer"
        }],
        "using": {
          "_obj": "adjustmentLayer",
          "type": {
            "_obj": "blackAndWhite",
            "blue": 20,
            "cyan": 60,
            "grain": 40,
            "magenta": 80,
            "presetKind": {
              "_enum": "presetKindType",
              "_value": "presetKindDefault"
            },
            "red": 40,
            "tintColor": {
              "_obj": "RGBColor",
              "blue": 179.00115966796876,
              "grain": 211.00067138671876,
              "red": 225.00045776367188
            },
            "useTint": false,
            "yellow": 60
          }
        }
      }
    ]
  },
  "outputs": [{
    "type": "image/jpeg",
    "storage": "<storage>",
    "href": "<SIGNED_POST_URL>"
  }]
}'
```

The actionJSON endpoint does support multiple inputs. If you would like to learn more about using multiple inputs with actionJSON, you can find this: [Multiple Inputs ActionJSON Example](../code-sample/index.md#executing-an-actionjson-with-multiple-inputs).

Take a look at this tutorial of this endpoint to learn more. Alternately you can read on in this section to walk through the process.

<Media slots="video" width="750" height="500"/>

<https://youtu.be/giFJ6qbez_I?feature=shared>

### Enable Developer Mode

If you haven't already enabled developer mode in your Photoshop app, follow these steps:

  * Select:
    * `Photoshop → Preferences → Plugins...` (For Mac)
   Or
    * `Edit → Preferences → Plugins...` (For Windows)
  * Select `Enable Developer Mode`
  * Quit Photoshop

  Enable this as a hidden feature if you are using Photoshop 23.4 (July 2022) or earlier. Execute the command below:

  For Mac

  ```bash
  echo "UXPEnableScriptingUtilities 1" >>  "/Users/$USER/Library/Preferences/Adobe Photoshop 2021 Settings/PSUserConfig.txt"
  ```

  For Windows Powershell
  
  ```bash
  echo  "UXPEnableScriptingUtilities 1" >> "C:\Users\$env:USERNAME\AppData\Roaming\Adobe\Adobe Photoshop 2021\Adobe Photoshop 2021 Settings\PSUserConfig.txt"
  ```

 At this point you can reopen your Photoshop app with developer mode enabled.

### Create New actionJSON

If you have developer mode enabled in Photoshop follow the instructions below. If you don't have developer mode enabled below please see the previous section.

* Open the Photoshop app
* Select `Settings | Plugins`
* Select `Development`
* Select `Record Action Commands...`
* Name your file and click `Save`. You can now make select actions in Photoshop such as resizing an image, adjusting hue and saturation and son on. Photoshop saves all of your actions in your new file.

Once you are done recording your action, you can stop recording and save:

* Select `Settings | Plugins`
* Select `Development`
* Click `Stop Action Recording`

Photoshop app saves your actions to the directory you chose when you named your file.

### Create actionJSON in Actions Panel

You can alternately create a new file in your Photoshop app's Action Panel:

* Go to `Windows | Actions`. The action panel opens.
* Select `New Action` to create a new action. You can alternately click `+` in the panel.
  ![alt image](./actions_panel_menu.png?raw=true "Original Image")
* Select your action from action set
* Select `Copy as Javascript`
* Paste it in any text editor.
* Modify the file to trim out the actions. An example is shown below in the code sample.

Now you can use the action in your Photoshop API payload

### Convert ATN files into actionJSON

This endpoint enables you to convert an .atn file to actionJSON format. This is the simplest and easiest way to create an actionJSON file.

### Convert ATN files into actionJSON with Photoshop

* Go to `Windows | Actions`. The action panel opens.
* Select `Load action`
* Choose the action you would like to convert to actionJSON
* Click on `copy as Javascript`
* Paste it in any text editor
* Modify the file to trim out the actions obj blocks An example is shown below in the code sample.

You can now use the action in your payload. Here is a code sample of Action JSON when you copy as Javascript from Photoshop:

```js
async function vignetteSelection() {
    let result;
    let psAction = require("photoshop").action;

    let command = [
        // Make snapshot
        {"_obj":"make","_target":[{"_ref":"snapshotClass"}],"from":{"_property":"currentHistoryState","_ref":"historyState"},"using":{"_enum":"historyState","_value":"fullDocument"}},
        // Feather
        {"descriptor": {"_obj":"feather","radius":{"_unit":"pixelsUnit","_value":5.0}}, "options": {"dialogOptions": "display"}},
        // Layer Via Copy
        {"_obj":"copyToLayer"},
        // Show current layer
        {"_obj":"show","null":[{"_enum":"ordinal","_ref":"layer","_value":"targetEnum"}],"toggleOptionsPalette":true},
        // Make layer
        {"_obj":"make","_target":[{"_ref":"layer"}]},
        // Fill
        {"_obj":"fill","mode":{"_enum":"blendMode","_value":"normal"},"opacity":{"_unit":"percentUnit","_value":100.0},"using":{"_enum":"fillContents","_value":"white"}},
        // Move current layer
        {"_obj":"move","_target":[{"_enum":"ordinal","_ref":"layer","_value":"targetEnum"}],"to":{"_enum":"ordinal","_ref":"layer","_value":"previous"}}
    ];
    result = await psAction.batchPlay(command, {});
}

async function runModalFunction() {
    await require("photoshop").core.executeAsModal(vignetteSelection, {"commandName": "Action Commands"});
}

await runModalFunction();
```

Modify the javascript file to trim out the actions.
Remove everything else from the javascript file and copy the array containing `_obj` from the `command` variable which will look something like below

```js
[      
 {"_obj":"make","_target":[{"_ref":"snapshotClass"}],"from":{"_property":"currentHistoryState","_ref":"historyState"},
 "using":{"_enum":"historyState","_value":"fullDocument"}},
 {"_obj":"feather","radius":{"_unit":"pixelsUnit","_value":5.0}},
 {"_obj":"copyToLayer"},
 {"_obj":"show","null":[{"_enum":"ordinal","_ref":"layer","_value":"targetEnum"}],"toggleOptionsPalette":true},
 {"_obj":"make","_target":[{"_ref":"layer"}]},
 {"_obj":"fill","mode":{"_enum":"blendMode","_value":"normal"},"opacity":{"_unit":"percentUnit","_value":100.0},"using":{"_enum":"fillContents","_value":"white"},
 {"_obj":"move","_target":[{"_enum":"ordinal","_ref":"layer","_value":"targetEnum"}],"to":{"_enum":"ordinal","_ref":"layer","_value":"previous"}}
]
```

More details about actionJSON can be found [here](https://developer.adobe.com/photoshop/uxp/2022/ps_reference/media/batchplay/)

## Smart Object

The Smart Object endpoint allows you to create and edit an embedded Smart Objects in a Photoshop file, or PSD file. The Smart Object that's replaced will be positioned within the bounding box of the original image. Whether the new image is larger or smaller than the original, it will adjust to fit within the original bounding box while preserving its aspect ratio. To alter the bounds of the replaced image, you can specify bounds parameters in the API call.

### Known Limitations

* If your document contains transparent pixels, (e.g some .png), you may not get consistent bounds.
* We currently do not support Linked Smart Objects.
* In order to update an embedded Smart Object that is referenced by multiple layers you need to update each of those layers in order for the Smart Object to be replaced in those layers.

Here is an [example](../code-sample/index.md#replacing-a-smartobject) of replacing a Smart Object within a layer.
For better performance, we rasterize our smart objects that are bigger than  2000 pixels * 2000 pixels.
For optimal processing, please make sure the embedded smart object that you want to replace only contains alphanumeric characters in it's name.

In this example, we generated a Smart Object within the "socks" layer and utilized the API to substitute the original image with a new pattern. This process facilitated the creation of two variations of the identical photograph.
![alt image](./smartobject_example.png?raw=true "Original Image")

## Text

The Edit Text endpoint supports editing one or more text layers within a PSD.

It enables users to:

* Format text properties such as antialias, orientation and be able to edit text contents. (Note: Changing only the text properties will not change any character/paragraph styling).
* Some of the key character properties that can be formatted include (but not limited to):
  * Text treatments such as strikethrough, underline, fontCaps.
  * Character size and color.
  * Line and character spacing through leading, tracking, autoKern settings.
* All the paragraph properties are supported.
* Use custom fonts when specified through the options.fonts section in the API request body.

### Usage Recommendations

* Ensure that the input file is a PSD and that it contains one or more text layers.
* Please refer to [Font Handling](../features/index.md#font-handling) and [Handle Missing Fonts](../features/index.md#handle-missing-fonts-in-the-document) for a better understanding.
* You can find a code sample [here.](../code-sample/index.md#making-a-text-layer-edit)

### Known Limitations

The API cannot automatically detect missing fonts in the layers. To prevent potential missing fonts from being replaced, please provide a href to the font(s) in the options.fonts section of the API. For more details on specifying custom fonts, please refer to the example section below.

In this example, the font on the original image was altered using the Text API, as depicted in the image on the left.
![alt image](./textlayer_example.png?raw=true "Original Image")

## Product Crop

The Product Crop endpoint facilitates smart cropping for images, automatically detecting the subject and ensuring it remains the focal point of the cropped image. You can identify the product and specify the desired padding for their cropped image. You can see some sample code [here.](../code-sample/index.md#applying-product-crop)

### Known Limitations

The current model is trained to return a crop that respects the salient object within an image. There is a current known issue that when a person or portrait is contained within a salient object, the model will crop with the person as the focal area rather than the salient object that contains it. This is problematic in the case of an item where an image of a person is contained within a design (i.e. a t-shirt, collectible or art). Rather than crop to the intended item, the service will crop to the person within the item.
We intend to correct this issue in future releases.

## DepthBlur

The DepthBlur API supports applying depth blur to your image. Depth Blur is part of the Neural Filters gallery in Photoshop. It allows you to target the area and range of blur in photos, creating wide-aperture depth of field blur effects. You may choose different focal points or remove the focal point and control the depth blur through manipulating the focal range slider. Setting focusSubject to true will select the most prominent subject in the image and apply depth blur around that subject.

You can find a code sample [here.](../code-sample/index.md#applying-depth-blur-neural-filter)

## Rendering and Conversions

This endpoint allows you to create a new PSD document and various renditions of different sizes. You can also convert any supported input file format to PSD, JPEG, TIFF, or PNG

* Create a new PSD document.
* Create a JPEG, TIFF or PNG rendition of various sizes.
* Request thumbnail previews of all renderable layers.
* Convert between any of the supported filetypes (PSD, JPEG, TIFF, PNG).

Here is an example of creating JPEG and PNG rendtions of a PSD document:
[Render PSD document](../code-sample/index.md#create-a-document-rendition)

## Layer level edits

* General layer edits
  * Edit the layer name.
  * Toggle the layer locked state.
  * Toggle layer visibility.
  * Move or resize the layer via it's bounds.
  * Delete layers.
* Adjustment layers
  * Add or edit an adjustment layer. The following types of adjustment layers are currently supported:
    * Brightness and Contrast.
    * Exposure.
    * Hue and Saturation.
    * Color Balance.
* Image/Pixel layers
  * Add a new pixel layer, with optional image.
  * Swap the image in an existing pixel layer.
* Shape layers
  * Resize a shape layer via it's bounds.

### Add, edit and delete layers

The `/documentOperations` API should primarily be used to make layer and/or document level edits to your PSD and then generate new renditions with the changes. You can pass in a flat array of only the layers that you wish to act upon, in the `options.layers` argument of the request body.
The layer name (or the layer id) will be used by the service to identify the correct layer to operation upon in your PSD.

The `add`, `edit`, `move` and `delete` blocks indicate the action you would like to be taken on a particular layer object. Any layer block passed into the API that is missing one of these attributes will be ignored.
The `add` and `move` blocks must also supply one of the attributes `insertAbove`, `insertBelow`, `insertInto`, `insertTop` or `insertBottom` to indicate where you want to move the layer to. More details on this can be found in the API reference.

**Note**: Adding a new layer does not require the ID to be included, the service will generate a new layer id for you.

Here are some examples of making various layer level edits.

* [Layer level editing](../code-sample/index.md#making-a-simple-edit)
* [Adding a new Adjustment Layer](../code-sample/index.md#adding-a-new-adjustment-layer)
* [Editing Image in a Pixel Layer](../code-sample/index.md#editing-a-pixel-layer)

## Text layers Edits

The Photoshop API currently supports creating and editing of Text Layer with different fonts, character styles and paragraph styles. The set of text attributes that can be edited is listed below:

* Edit the text contents
* Change the font (See the `Fonts` section for more info)
* Edit the font size
* Change the font color in the following formats: rgb, cmyk, gray, lab
* Edit the text orientation (horizontal/vertical)
* Edit the paragraph alignment (left, center, right, justify, justifyLeft, justifyCenter, justifyRight)

We also have an example of making a simple text layer edit.

[Text layer Example Code](../code-sample/index.md#edit-text-layers)

### Font handling

In order to be able to correctly operate on text layers in the PSD, the corresponding fonts needed for these layers will need to be available when the server is processing the PSD. These include fonts from the following cases:

1. The font that is in the text layer being edited, but the font itself is not being changed
2. If the font in a text layer is being changed to a new font

While referencing fonts in the API request, please ensure that the correct Postscript name for that font is used. Referencing to that font with any other name will result in the API treating this as a missing font.

The Photoshop API supports using the following category of fonts:

* You can find a list of currently supported fonts [here](../features/index.md#photoshop-cc)
* Custom/Other Fonts: These are the fonts that are either owned by you or the ones that only you are authorized to use.
  To use a custom font you must include an href to the font in your request. Look at the `options.fonts` section of the API docs for more information.
  For including an href to the font in your request, please ensure the font file name to be in this format: `<font_postscript_name>.<ext>`, when it is being uploaded in your choice of storage. A sample `options.fonts` section will look like so:

  ```json
  {
    "storage": "external",
    "href": "<Storage URL to OpenSansCondensed-Light.ttf>"
  }
  ```

  **Note:** This also applies to any other font present in the document which is not to be found in the first 2 categories above.

Here is an example usage of a custom font: [Custom font](../code-sample/index.md#custom-font-in-a-text-layer)

#### Handle missing fonts in the document.

The API provides two options to control the behavior when there are missing fonts, as the request is being processed:

Specify a global font which would act as a default font for the current request: The `globalFont` field in the `options` section of the request can be used to specify the full postscript name of this font.
For any textLayer edit/add operation, if the font used specifically for that layer is missing, this font will be used as the default. If the global font itself is missing, then the action to be taken will be dictated by the `manageMissingFonts` options as explained here in the next bullet point.

**Note**: If using an OAuth integration, Adobe Fonts can be used as a global font as well. If the global font is a custom font, please upload the font to one of the cloud storage types that is supported and specify the `href` and `storage` type in the `options.fonts` section of the request.

Specify the action to be taken if one or more fonts required for the add/edit operation(s) are missing: The `manageMissingFonts` field in the `options` section of the request can be used to specify this action. It can accept one of the following 2 values:

* `fail` to force the request/job to fail
* `useDefault` to use our system designated default font, which is: `ArialMT`

In this example we show you how to [andle missing fonts](../code-sample/index.md#dictating-actions-for-missing-fonts) using the `manageMissingFonts` and `globalFont` options.

### Limitations

Most of the text attributes retain their respective original values. There are some attributes however that do not retain their original values. For example (and not limited to): tracking, leading, kerning.

### Supported Fonts

This is a list of all of the supported Postscript fonts for Photoshop API.

### Photoshop CC

|                                   |
|---------------------------------- |
| AcuminVariableConcept             |
| AdobeArabic-Bold                  |
| AdobeArabic-BoldItalic            |
| AdobeArabic-Italic                |
| AdobeArabic-Regular               |
| AdobeDevanagari-Bold              |
| AdobeDevanagari-BoldItalic        |
| AdobeDevanagari-Italic            |
| AdobeDevanagari-Regular           |
| AdobeFanHeitiStd-Bold             |
| AdobeGothicStd-Bold               |
| AdobeGurmukhi-Bold                |
| AdobeGurmukhi-Regular             |
| AdobeHebrew-Bold                  |
| AdobeHebrew-BoldItalic            |
| AdobeHebrew-Italic                |
| AdobeHebrew-Regular               |
| AdobeHeitiStd-Regular             |
| AdobeMingStd-Light                |
| AdobeMyungjoStd-Medium            |
| AdobePiStd                        |
| AdobeSongStd-Light                |
| AdobeThai-Bold                    |
| AdobeThai-BoldItalic              |
| AdobeThai-Italic                  |
| AdobeThai-Regular                 |
| CourierStd                        |
| CourierStd-Bold                   |
| CourierStd-BoldOblique            |
| CourierStd-Oblique                |
| EmojiOneColor                     |
| KozGoPr6N-Bold                    |
| KozGoPr6N-Medium                  |
| KozGoPr6N-Regular                 |
| KozMinPr6N-Regular                |
| MinionPro-Regular                 |
| MinionVariableConcept-Italic      |
| MinionVariableConcept-Roman       |
| MyriadArabic-Bold                 |
| MyriadArabic-BoldIt               |
| MyriadArabic-It                   |
| MyriadArabic-Regular              |
| MyriadHebrew-Bold                 |
| MyriadHebrew-BoldIt               |
| MyriadHebrew-It                   |
| MyriadHebrew-Regular              |
| MyriadPro-Bold                    |
| MyriadPro-BoldIt                  |
| MyriadPro-It                      |
| MyriadPro-Regular                 |
| MyriadVariableConcept-Italic      |
| MyriadVariableConcept-Roman       |
| NotoSansKhmer-Regular             |
| NotoSansLao-Regular               |
| NotoSansMyanmar-Regular           |
| NotoSansSinhala-Regular           |
| SourceCodeVariable-Italic         |
| SourceCodeVariable-Roman          |
| SourceSansVariable-Italic         |
| SourceSansVariable-Roman          |
| SourceSerifVariable-Roman         |
| TrajanColor-Concept               |

## Document level edits

* Crop a PSD
* Resize a PSD

## Artboards

* Show artboard information in the JSON Manifest
* Create a new artboard from multiple input psd's

## Remove Background

The Remove Background endpoint can recognize the primary subject within an image and eliminate the background, providing the subject as the output. You can see a code sample [here.](../code-sample/index.md#remove-background).<br />

Example of Remove Background with a sample image.
![alt image](./imagecutout_cutout_example.png?raw=true "Original Image")

## Create Mask

This endpoint allows you create a greyscale mask png file that you can composite onto the original image (or any other). You can find a code sample [here.](../code-sample/index.md#generate-image-mask).<br />

Example of Image mask with a sample image.
![alt image](./imagecutout_mask_example.png?raw=true "Original Image")

## Customized Workflow

You can make a customized workflow by chaining different endpoints together. [Here](../code-sample/index.md#generate-remove-background-result-as-photoshop-path) is an example using the Remove Background endpoint.

## Webhooks through Adobe I/O Events

Adobe I/O Events offers the possibility to build an event-driven application, based on events originating from Photoshop. To start listening for events, your application needs to register a webhook URL, specifying the Event Types to receive. Whenever a matching event gets triggered, your application is notified through an HTTP POST request to the webhook URL.
The Event Provider for the Photoshop API is `Imaging API Events`.
This event provider has two event types:

1. `Photoshop API events`

As the names indicate, these event types represent events triggered by the individual APIs.

### Registering your application to our Event Provider

#### Prerequisites needed to use the Event Provider

1. In order to use the Adobe I/O Events you will need to create a project on Adobe I/O Console.
2. You can follow the steps listed in [Getting Started](../../guides/get-started.md) page if you haven't created one yet.
3. Create a [Webhook application](https://www.adobe.io/apis/experienceplatform/events/docs.html#!adobedocs/adobeio-events/master/intro/webhooks_intro.md)

You can find a sample NodeJS application [here](https://github.com/AdobeDocs/cis-photoshop-api-docs/tree/main/sample-code/webhook-sample-app).

#### Registering the Webhook

Once the above prerequisites are met, you can now proceed to register the webhook to the service integration. The steps to do that can be found  [here](https://www.adobe.io/apis/experienceplatform/events/docs.html#!adobedocs/adobeio-events/master/intro/webhooks_intro.md#your-first-webhook).
After the webhook has been successfully registered, you will start to receive the events for any submitted job that either succeeded or failed, from the Event Types selected. This eliminates the need for your application to poll for the status of the job using the jobID. Examples can be found [here](../code-sample/index.md#triggering-an-event-from-the-apis)
