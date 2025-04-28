import Foundation
import React

@objc(DBSFetchModule)
class DBSFetchModule: NSObject {
    @objc static func requiresMainQueueSetup() -> Bool { false }

    @objc(fetch:resolver:rejecter:)
    func fetch(
        urlString: String,
        resolver resolve: @escaping RCTPromiseResolveBlock,
        rejecter reject: @escaping RCTPromiseRejectBlock
    ) {
        DBSFetch.shared.fetch(urlString: urlString) { result in
            switch result {
            case .success(let data):
                let resultStr = String(data: data, encoding: .utf8) ?? ""
                resolve(resultStr)
            case .failure(let error):
                reject("fetch_error", error.localizedDescription, error)
            }
        }
    }
}
