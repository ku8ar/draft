#import "DBSFetchModule.h"

@implementation DBSFetchModule

RCT_EXPORT_MODULE();

RCT_REMAP_METHOD(fetch,
                 fetchWithURLString:(NSString *)urlString
                 resolver:(RCTPromiseResolveBlock)resolve
                 rejecter:(RCTPromiseRejectBlock)reject)
{
    NSURL *url = [NSURL URLWithString:urlString];
    if (!url) {
        reject(@"invalid_url", @"Invalid URL", nil);
        return;
    }
    NSURLSessionDataTask *task = [[NSURLSession sharedSession] dataTaskWithURL:url
                                                             completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
        if (error) {
            reject(@"fetch_error", error.localizedDescription, error);
            return;
        }
        if (!data) {
            reject(@"no_data", @"No data returned", nil);
            return;
        }
        NSString *resultStr = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
        if (!resultStr) resultStr = @"";
        resolve(resultStr);
    }];
    [task resume];
}

@end
