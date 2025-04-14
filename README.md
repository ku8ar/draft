post_install do |installer|
  config = use_native_modules!

  react_modular = %w[
    React
    React-Core
    React-CoreModules
    React-RCTBridge
    ReactCommon
    RCTTypeSafety
    RCTRequired
  ]

  installer.pod_targets.each do |pod|
    if react_modular.include?(pod.name)
      pod.specs.each do |spec|
        spec.instance_variable_set(:@modular_headers, true)
      end
    end
  end

  react_native_post_install(
    installer,
    config[:reactNativePath],
    :mac_catalyst_enabled => false
  )
end
