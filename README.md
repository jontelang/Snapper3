# Snapper 3 extensions

To add an action into Snapper 3, you as a developer needs to do the following:

- Create an object conforming to the provided `Snapper3Plugin` protocol
- Provide icons and descriptions
- Register it into Snapper 3's plugin manager during runtime, after the system as loaded (optimally after a small delay as well)

## Requirements

- This will only work on version 1.1.1 and up

## Protocol

```objective-c
@protocol Snapper3Plugin <NSObject>

@required
-(BOOL)removeSnapAfterProcessing; // If the snap is keps on screen or not after long holding on it. Normally just return YES here.
-(UIImage*)image; // This is the image in the 'action' bar.
-(UIImage*)imageForMenuAndSettings; // This is an image that will show in in settings AND when a user long presses the snap. You might want to return different ones for dark/light mode.
-(NSString*)pluginIdentifier; // A UNIQUE identifier, this is internally used so make sure it is unique.
-(BOOL)shouldRegister; // Wether the plugin should be visible in settings or in snapper at all. For example, if your plugin requires a certain app to be installed, you could return NO here if that app is not installed on the device. Or if only certain iOS versions are supported, you could return NO.
-(BOOL)showInSettings; // Wether it should show up in settings app or not. Just return YES here.
-(BOOL)disabledInitially; // Weather the tweak is enabled or not right after an install.
-(NSString*)name; // The name that will show up in the menu and settings. Should NOT be nil.
-(NSString*)info; // A description of what the plugin does. Can be nil.
-(NSString*)tweakIdentifier; // This is a package identier and will open a deep link to Sileo if provided. Can be nil.
-(NSString*)email; // An email which a user can contact you through. Can be nil.
-(NSString*)twitter; // Your twitter handle. Can be nil.
-(NSString*)website; /// Keep https://. Can be nil.

@optional
-(void)processImage:(UIImage*)image; // When your plugin action is pressed, this is what is called. Do what you want with the image here.

@end
```

And this is the interface Snapper 3 provides as of 1.1

```objective-c
@interface Snapper3PluginManager: NSObject
+(Snapper3PluginManager*)sharedInstance;
-(void)registerPlugin:(id<Snapper3Plugin>)plugin;
@end
```

## Notes

- Keep dark / light mode in mind
- Follow the comments above
- Get in touch if something doesn't work

## When to register

You should register your plugin into Snapper 3 after the device starts up, see the example below of when you can do it.

## Example plugin

This plugin is the plugin which is actually bundled into Snapper 3 1.1. It saves an image into the Photos app and opens a deep link to an app.

```objective-c
@interface SnapperDAMAAppPlugin: NSObject <Snapper3Plugin> @end

---

@implementation SnapperDAMAAppPlugin

-(void)processImage:(UIImage*)image { 
    UIImageWriteToSavedPhotosAlbum(image, nil, nil, nil);
    [[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"dama://redactLast"] options:@{} completionHandler:nil];
}

-(BOOL)removeSnapAfterProcessing { return YES; }
-(NSString*)name { return @"DAMA shortcut"; }
-(NSString*)info { return @"This shortcut will open the app DAMA. You need to install DAMA from the App Store for this to work!"; }
-(UIImage*)image { return PLUGIN_IMAGE_DAMA; } // Use your own bundle for the image
-(UIImage*)imageForMenuAndSettings { return PLUGIN_IMAGE_DAMA; } // Use your own bundle for the image
-(NSString*)pluginIdentifier { return @"com.jontelang.snapper3.plugin.dama"; }
-(NSString*)developer { return nil; } 
-(NSString*)email { return nil; } 
-(NSString*)tweakIdentifier { return nil; } 
-(NSString*)twitter { return nil; } 
-(NSString*)website { return nil; } 
-(BOOL)showInSettings { return YES; }
-(BOOL)shouldRegister { return YES; }
-(BOOL)disabledInitially { return YES; } // This is just because not all wants to use this
@end
```

And here is how you can register it into Snapper 3

```objective-c
static inline void initializeTweak(CFNotificationCenterRef center, void *observer, CFStringRef name, const void *object, CFDictionaryRef userInfo) {
    // Wait a bit to ensure Snapper 3 is loaded
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(5.0f * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        // Load the Snapper 3 plugin manager during runtime
        Snapper3PluginManager *s3pm = [objc_getClass("Snapper3PluginManager") sharedInstance];
        // Register the plugin
        [s3pm registerPlugin:[[SnapperDAMAAppPlugin alloc] init]];
    });
}

%ctor {
    CFNotificationCenterAddObserver(
        CFNotificationCenterGetDarwinNotifyCenter(), 
        NULL, 
        &initializeTweak, 
        CFSTR("SBSpringBoardDidLaunchNotification"), 
        NULL, 
        CFNotificationSuspensionBehaviorDeliverImmediately);
}
```

And that should be it.
