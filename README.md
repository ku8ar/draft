//
//  OnfidoSdk.swift
//
//  Copyright © 2016-2025 Onfido. All rights reserved.
//

import Foundation
import Onfido
import UIKit

@objc(OnfidoSdkBridge)
public final class OnfidoSdkBridge: NSObject {
    @objc public static let shared = OnfidoSdkBridge()

    private let onfidoFlowBuilder = OnfidoFlowBuilder()
    private let configParser = OnfidoConfigParser()
    private var callbackTypes: [CallbackType] = []

    private lazy var encryptedBiometricTokenHandlerReceiver = EncryptedBiometricTokenHandlerReceiver(
        withTokenRequestedCallback: { [weak self] result in
            self?.sendEvent(name: "onTokenRequested", body: result)
        },
        andTokenGeneratedCallback: { [weak self] result in
            self?.sendEvent(name: "onTokenGenerated", body: result)
        }
    )

    private enum CallbackType {
        case media
        case encryptedBiometricToken
    }

    @objc public func start(with config: NSDictionary,
                            resolve: @escaping (Any?) -> Void,
                            reject: @escaping (String, String, NSError?) -> Void) {
        DispatchQueue.main.async {
            self.run(withConfig: config, resolve: resolve, reject: reject)
        }
    }

    private func run(withConfig config: NSDictionary,
                     resolve: @escaping (Any?) -> Void,
                     reject: @escaping (String, String, NSError?) -> Void) {
        do {
            let onfidoConfig = try configParser.parse(config)
            let appearanceFilePath = Bundle.main.path(forResource: "colors", ofType: "json")
            let appearance = try loadAppearanceFromFile(filePath: appearanceFilePath)

            if #available(iOS 12.0, *), let theme = onfidoConfig.theme {
                switch theme {
                case .dark: appearance.setUserInterfaceStyle(.dark)
                case .light: appearance.setUserInterfaceStyle(.light)
                case .automatic: appearance.setUserInterfaceStyle(.unspecified)
                }
            }

            let mediaCallback = callbackTypes.contains(.media)
                ? CallbackReceiver(withCallback: { [weak self] result in
                    self?.sendEvent(name: "onfidoMediaCallback", body: result)
                })
                : nil

            let encryptedBiometricTokenHandler = callbackTypes.contains(.encryptedBiometricToken)
                ? encryptedBiometricTokenHandlerReceiver
                : nil

            let onfidoFlow = try onfidoFlowBuilder.build(
                with: onfidoConfig,
                appearance: appearance,
                customMediaCallback: mediaCallback,
                customEncryptedBiometricTokenHandler: encryptedBiometricTokenHandler
            )

            onfidoFlow.with(responseHandler: { response in
                switch response {
                case .success(let results):
                    resolve(createResponse(results))
                case .error(let error):
                    reject("onfido_error", "Error during flow", error as NSError)
                case .cancel(let reason):
                    switch reason {
                    case .deniedConsent:
                        reject("deniedConsent", "User denied consent.", nil)
                    case .userExit:
                        reject("userExit", "User canceled flow.", nil)
                    default:
                        reject("userExit", "User canceled flow via unknown method.", nil)
                    }
                @unknown default:
                    reject("unknown", "Unknown error occurred", nil)
                }
            })

            guard let window = UIApplication.shared.windows.first,
                  let topVC = window.rootViewController?.findTopMostController() else {
                reject("present_error", "Unable to locate presenting view controller", nil)
                return
            }

            try onfidoFlow.run(from: topVC, presentationStyle: .fullScreen, animated: true)

        } catch let error as NSError {
            reject("exception", error.localizedDescription, error)
        }
    }

    // MARK: - Event forwarding

    private func sendEvent(name: String, body: Any) {
        NotificationCenter.default.post(name: Notification.Name(name), object: body)
        // lub wywołaj delegata, jeśli robisz most ręczny
    }

    // MARK: - Public API

    @objc public func supportedEvents() -> [String] {
        return ["onfidoMediaCallback", "onTokenRequested", "onTokenGenerated"]
    }

    @objc public func withMediaCallbacksEnabled() {
        callbackTypes.append(.media)
    }

    @objc public func withBiometricTokenCallback() {
        callbackTypes.append(.encryptedBiometricToken)
    }

    @objc public func provideBiometricToken(_ biometricToken: String) {
        encryptedBiometricTokenHandlerReceiver.provide(encryptedBiometricToken: biometricToken)
    }
}

// MARK: - Helpers

public extension UIViewController {
    func findTopMostController() -> UIViewController? {
        var top = self
        while let presented = top.presentedViewController {
            top = presented
        }
        return top
    }
}

extension Dictionary where Key == String {
    func getColor(withName name: String, fallback: UIColor) -> UIColor {
        if let colorString = self[name] as? String {
            return UIColor.from(hex: colorString)
        } else {
            return fallback
        }
    }
}
