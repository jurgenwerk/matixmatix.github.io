---
category: posts
layout: single
mainTopic: "programming"
title: "Making emojis work in Headless Chrome using AWS Lambda and Puppeteer"
---

This is a small tutorial on how to make emojis work when rendering emoji content
using Headless Chrome in an AWS Lambda function.

In my case, I needed to convert HTML to PDF, but this can also be used for rendering images, scraping, or doing automated testing.

The components used:

- AWS Lambda (with layers)
- Serverless framework
- Headless Chrome
- Puppeteer

By default, the emojis won't render in Headless Chrome when it's being run inside a Lambda function, and what you get is just squares instead of emojis.

To solve this, you have to provide an emoji font which the Headless Chrome can use to render emojis.

### Structure

Create a directory like this:

```
/lambda-function
  |_ layers
    |_ chrome_aws_lambda.zip
    |_ custom_fonts.zip
  handler.js
  package.json
  serverless.yml
```

### Layers

This uses 2 Lambda layers; one for Headless Chrome and one for the emoji font.

Download `chrome_aws_lambda.zip` from [here](https://github.com/shelfio/chrome-aws-lambda-layer), and for the custom_fonts.zip, do the following:

1. Download the [NotoColorEmoji](https://github.com/googlefonts/noto-emoji/blob/master/fonts/NotoColorEmoji.ttf) font
2. Create a folder `custom_fonts`, put the font inside it and compress the folder into a `custom_fonts.zip``

Put both zip files into the `layers` folder.

### Serverless config

Here's how the [Serverless](https://www.serverless.com/) deploy file looks like:

```yaml
service: my-pdf-converter-service

provider:
  name: aws
  runtime: nodejs12.x
  timeout: 15
  region: eu-west-1
  iamRoleStatements:
    - Effect: Allow
      Action:
        - s3:PutObject
      Resource: "arn:aws:s3:::my-bucket/environment-${opt:stage}/*"

functions:
  convertToPdfAndUpload:
    handler: handler.convertToPdfAndUpload
    layers:
      - { Ref: HeadlessChromeLambdaLayer }
      - { Ref: CustomFontsLambdaLayer }

layers:
  HeadlessChrome:
    name: HeadlessChrome
    package:
      artifact: layers/chrome_aws_lambda.zip
  CustomFonts:
    name: CustomFonts
    package:
      artifact: layers/custom_fonts.zip
```

### Dependencies

Two dependencies are needed:

```json
{
  "name": "my-lambda-functions",
  "version": "1.0.0",
  "description": "AWS Lambda function for converting HTML to PDF",
  "main": "handler.js",
  "dependencies": {
    "chrome-aws-lambda": "^5.5.0",
    "puppeteer-core": "^5.5.0"
  }
}
```

Make sure to install them using `npm install`, or `yarn install`.

### The function

The key here is to import the font from the layer, using `await chromium.font("/opt/custom_fonts/NotoColorEmoji.ttf");`.

```javascript
const chromium = require("chrome-aws-lambda");
const AWS = require("aws-sdk");

module.exports.convertToPdfAndUpload = async function (
  event,
  context,
  callback
) {
  const { html, key, bucket } = event;

  await chromium.font("/opt/custom_fonts/NotoColorEmoji.ttf");

  browser = await chromium.puppeteer.launch({
    args: chromium.args,
    defaultViewport: chromium.defaultViewport,
    executablePath: await chromium.executablePath,
    headless: true,
  });

  const page = await browser.newPage();
  await page.setContent(html);

  const pdf = await page.pdf({ format: "A4" });

  await browser.close();

  const uploadParams = {
    Bucket: bucket,
    Key: key,
    Body: pdf,
    ContentType: "application/pdf",
  };

  const obj = await new AWS.S3().upload(uploadParams).promise();

  callback(null, { url: obj["Location"] });
};
```

Now deploy it using `serverless deploy` and enjoy your AWS Lambda rendered emojis.
