
# Push Notifications with Images in Notification Center

This guide demonstrates how to send and display push notifications with images in the iOS notification center, along with customizations for handling media attachments.

## Features
- Display an image in the notification center.
- Customize notifications with media attachments (images).
- Download the image from a URL and attach it to the notification.

## Steps to Implement

### 1. Modify Push Notification Payload

In your server-side push notification payload, include the URL of the image you want to display in the notification center. The image should be hosted on a server that allows access via HTTPS.

#### Example Payload:
```json
{
  "aps": {
    "alert": {
      "title": "New Update",
      "body": "Check out the latest feature!"
    },
    "badge": 1,
    "sound": "default"
  },
  "media-url": "https://example.com/path/to/image.jpg"
}
```

### 2. Handle Image in iOS App

In your iOS app, implement the `userNotificationCenter(_:didReceive:withCompletionHandler:)` method to download the image from the provided URL and attach it to the notification.

#### Code Implementation:
```swift
import UserNotifications

// This method is triggered when a notification is received
func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive notification: UNNotification, withCompletionHandler completionHandler: @escaping () -> Void) {
    // Check if the media-url is included in the payload
    if let mediaURL = notification.request.content.userInfo["media-url"] as? String {
        // Download the image from the URL
        downloadImage(from: mediaURL) { image in
            // Attach the image to the notification
            if let attachment = createImageAttachment(from: image) {
                let mutableContent = notification.request.content.mutableCopy() as! UNMutableNotificationContent
                mutableContent.attachments = [attachment]
                
                let request = UNNotificationRequest(identifier: notification.request.identifier, content: mutableContent, trigger: nil)
                UNUserNotificationCenter.current().add(request, withCompletionHandler: nil)
            }
        }
    }
    completionHandler()
}

// Download image from URL
func downloadImage(from url: String, completion: @escaping (UIImage?) -> Void) {
    guard let imageURL = URL(string: url) else {
        completion(nil)
        return
    }
    
    let task = URLSession.shared.dataTask(with: imageURL) { data, _, _ in
        if let data = data, let image = UIImage(data: data) {
            completion(image)
        } else {
            completion(nil)
        }
    }
    task.resume()
}

// Create image attachment for notification
func createImageAttachment(from image: UIImage) -> UNNotificationAttachment? {
    guard let imageData = image.jpegData(compressionQuality: 1.0) else { return nil }
    let tempURL = FileManager.default.temporaryDirectory.appendingPathComponent(UUID().uuidString + ".jpg")
    
    do {
        try imageData.write(to: tempURL)
        return try UNNotificationAttachment(identifier: "image", url: tempURL, options: nil)
    } catch {
        return nil
    }
}
```

### 3. Customize the Image Display

You can control how the image is displayed in the notification center:
- **Thumbnail Size**: The system automatically generates a thumbnail for the image attachment. You can control its display properties by specifying options like `.thumbnailHidden` in `UNNotificationAttachmentOptions`.
- **Image Type**: If you want to control how the media is displayed (as a photo, video, etc.), you can set the `UNNotificationAttachment`'s `options` argument.

### 4. Testing Push Notifications with Images

To test push notifications with images:
1. Obtain the device token by registering the device for push notifications.
2. Send a test push notification from your server with the appropriate payload including the `media-url` key.
3. Make sure that the app handles the image and displays it in the notification center when the notification is received.

## Notes:
- Make sure the image is accessible via HTTPS.
- The maximum payload size for push notifications is 4 KB, so make sure the image URL does not cause the payload to exceed this limit.
- This solution requires the `UserNotifications` framework.

## Additional Resources
- [Apple Push Notification Service Documentation](https://developer.apple.com/documentation/usernotifications)
- [UNNotificationAttachment Class](https://developer.apple.com/documentation/usernotifications/unnotificationattachment)
