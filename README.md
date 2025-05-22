#import "DBSFetchModule.h"

@implementation DBSFetchModule

RCT_EXPORT_MODULE(DBSFetchModule)

RCT_EXPORT_METHOD(fetch:(NSDictionary *)request
                  resolver:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)
{
  NSString *urlString = request[@"url"];
  if (![urlString isKindOfClass:[NSString class]]) {
    reject(@"invalid_url", @"URL must be a string", nil);
    return;
  }

  NSURL *url = [NSURL URLWithString:urlString];
  if (!url) {
    reject(@"invalid_url", @"Invalid URL format", nil);
    return;
  }

  NSString *method = [request[@"method"] uppercaseString] ?: @"GET";
  NSDictionary *headers = request[@"headers"] ?: @{};
  NSString *bodyString = request[@"body"]; // optional, expected as string

  NSMutableURLRequest *urlRequest = [NSMutableURLRequest requestWithURL:url];
  [urlRequest setHTTPMethod:method];

  for (NSString *key in headers) {
    id value = headers[key];
    if ([value isKindOfClass:[NSString class]]) {
      [urlRequest setValue:value forHTTPHeaderField:key];
    }
  }

  if (bodyString && [bodyString isKindOfClass:[NSString class]]) {
    NSData *bodyData = [bodyString dataUsingEncoding:NSUTF8StringEncoding];
    [urlRequest setHTTPBody:bodyData];
  }

  NSURLSession *session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration]];
  NSURLSessionDataTask *task = [session dataTaskWithRequest:urlRequest
                                          completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {

    if (error) {
      reject(@"network_error", error.localizedDescription, error);
      return;
    }

    NSHTTPURLResponse *httpResponse = (NSHTTPURLResponse *)response;
    NSDictionary *responseHeaders = httpResponse.allHeaderFields;
    NSInteger statusCode = httpResponse.statusCode;
    NSString *body = data ? [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding] : @"";

    resolve(@{
      @"status": @(statusCode),
      @"headers": responseHeaders,
      @"body": body
    });
  }];

  [task resume];
}

@end
