import Foundation

@objc public class RNBridgeSafeFetch: NSObject {
    @objc public static let shared = RNBridgeSafeFetch()
    private override init() {}

    private var session: URLSession = .shared

    @objc public func configure(with session: URLSession) {
        self.session = session
    }

    @objc public func fetch(
        _ urlString: String,
        resolver resolve: @escaping RCTPromiseResolveBlock,
        rejecter reject: @escaping RCTPromiseRejectBlock
    ) {
        guard let url = URL(string: urlString) else {
            reject("invalid_url", "Invalid URL", nil)
            return
        }
        let task = session.dataTask(with: url) { data, response, error in
            if let error = error {
                reject("fetch_error", error.localizedDescription, error)
                return
            }
            guard let data = data else {
                reject("no_data", "No data returned", nil)
                return
            }
            let result = String(data: data, encoding: .utf8) ?? ""
            resolve(result)
        }
        task.resume()
    }
}
