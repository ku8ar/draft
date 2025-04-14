NSDictionary+Color.h

#import <UIKit/UIKit.h>

@interface NSDictionary (Color)

- (UIColor *)getColorWithName:(NSString *)name fallback:(UIColor *)fallback;

@end


---- NSDictionary+Color.m


#import "NSDictionary+Color.h"

@implementation NSDictionary (Color)

- (UIColor *)getColorWithName:(NSString *)name fallback:(UIColor *)fallback {
    id colorString = self[name];
    if (![colorString isKindOfClass:[NSString class]]) {
        return fallback;
    }

    NSString *hex = [(NSString *)colorString stringByReplacingOccurrencesOfString:@"#" withString:@""];
    if (hex.length != 6) {
        return fallback;
    }

    unsigned int rgbValue = 0;
    NSScanner *scanner = [NSScanner scannerWithString:hex];
    [scanner scanHexInt:&rgbValue];

    CGFloat red = ((rgbValue >> 16) & 0xFF) / 255.0;
    CGFloat green = ((rgbValue >> 8) & 0xFF) / 255.0;
    CGFloat blue = (rgbValue & 0xFF) / 255.0;

    return [UIColor colorWithRed:red green:green blue:blue alpha:1.0];
}

@end
