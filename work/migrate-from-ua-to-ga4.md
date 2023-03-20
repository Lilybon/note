# Migrate from UA to GA4

## Add a Google Analytics 4 property

1. Search your **UA ID** in search bar and navigate to the UA dashboard.

   ![add ga4 property - step1](https://i.imgur.com/bkewR80.png)

2. Click **Admin** at the bottom left corner of menu.

   ![add ga4 property - step2](https://i.imgur.com/jBGblXj.png)

3. Click **GA4 Setup Asistant**.

   ![add ga4 property - step3](https://i.imgur.com/qCpkAA7.png)

4. Click **Get Started**

   ![add ga4 property - step4](https://i.imgur.com/Q8eqIx0.png)

5. Click **Create and Continue**.

   ![add ga4 property - step5](https://i.imgur.com/MD36tq8.png)

6. Select **Install a Google tag** and click **Next**.

   ![add ga4 property - step6](https://i.imgur.com/mQbdrGK.png)

7. Select **Install manually** tab and click **Done**.

   ![add ga4 property - step7](https://i.imgur.com/t8wteXE.png)

8. Click **Go to your GA4 property**.

   ![add ga4 property - step8](https://i.imgur.com/u4GX1rY.png)

9. Awesome! Now you have a new GA4 dashboard.

   ![add ga4 property - step9](https://i.imgur.com/ZYqAboT.png)

## Install Your Google Tag

1. Get your gtag id in dashboard homepage.

   ![install your google tag - step1](https://i.imgur.com/fXQpDM8.png)

2. Replace `GA_TRACKING_ID` with your own gtag id.

   ```html
   <!-- Google tag (gtag.js) -->
   <script
     async
     src="https://www.googletagmanager.com/gtag/js?id=GA_TRACKING_ID"
   ></script>
   <script>
     window.dataLayer = window.dataLayer || []
     function gtag() {
       window.dataLayer.push(arguments)
     }
     gtag('js', new Date())

     gtag('config', 'GA_TRACKING_ID')
   </script>
   ```

3. Feel free with enhanced measurement like scroll, pageview and so on, check [[GA4] Enhanced event measurement](https://support.google.com/analytics/answer/9216061?hl=en) for more detail.

   ![install your google tag - step3](https://media.tenor.com/bxHQ4KcM8eMAAAAC/magic-meme.gif)

4. To send Google Analytics events on a web page, use the gtag.js event command with the following syntax:

   ```javascript
   gtag('event', '<action>', {
     event_category: '<category>',
     event_label: '<label>',
   })
   ```

5. See more information for how to send data through gtag in [在網站上加入 gtag.js](https://developers.google.com/analytics/devguides/collection/gtagjs?hl=zh-tw).

## Testing

1. Go to [Google Tag Assistant](https://tagassistant.google.com/) and download the chrome plugin.
2. Click **Add domain**.

   ![testing - step2](https://i.imgur.com/XY5J93Y.png)

3. Paste your **website's URL** and then press **Connect**.

   ![testing - step3](https://i.imgur.com/RB1hDXt.png)

4. Tag Assistant will open a testing tab for you.

   ![testing - step4](https://i.imgur.com/lbkSBbS.png)

5. Go back to Tag Assistant, Click **Continue** to start testing gtag.
   ![testing - step5](https://i.imgur.com/XFP2QL8.png)
6. Switch to specific **gtag id**, and event output displays below.

   ![testing - step6](https://i.imgur.com/5AT8sJT.png)

7. Trigger any custom event, and the result should be updated in the panel, click it for more detail.

   ![testing - step7](https://i.imgur.com/XNi80yg.png)

8. Check whether event payload is correct.

   ![testing - step8](https://i.imgur.com/UqxvTHY.png)

## Reference

- [[GA4] Add a Google Analytics 4 property (to a site that already has Analytics)](https://support.google.com/analytics/answer/9744165#zippy=%2Cin-this-article)
- [[GA4] Setup Assistant](https://support.google.com/analytics/answer/10312255)
- [[GA4] Navigate to your account and property](https://support.google.com/analytics/answer/12813202)
