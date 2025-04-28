import Foundation

@objc public class RNBridgeSafeFetch: NSObject {
    @objc public static let shared = RNBridgeSafeFetch()
    private override init() {}

    private var session: URLSession = .shared

    @objc public func configure(with session: URLSession) {
        self.session = session
    }

    // Interfejs dla bridge'a
    public func fetch(
        urlString: String,
        completion: @escaping (Result<Data, Error>) -> Void
    ) {
        guard let url = URL(string: urlString) else {
            completion(.failure(NSError(domain: "RNBridgeSafeFetch", code: 1, userInfo: [NSLocalizedDescriptionKey: "Invalid URL"])))
            return
        }
        let task = session.dataTask(with: url) { data, response, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            guard let data = data else {
                completion(.failure(NSError(domain: "RNBridgeSafeFetch", code: 2, userInfo: [NSLocalizedDescriptionKey: "No data returned"])))
                return
            }
            completion(.success(data))
        }
        task.resume()
    }
}
