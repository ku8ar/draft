post_install do |installer|
  react_native_post_install(installer, config[:reactNativePath])

  is_bridge_target = installer.aggregate_targets.any? { |t| t.name.downcase.include?('bridge') }

  installer.pods_project.targets.each do |target|
    if is_bridge_target && target.name.end_with?('_privacy')
      target.remove_from_project
      next
    end

    # 🧯 Wyłącz BUILD_LIBRARY_FOR_DISTRIBUTION dla głównych i pobocznych targetów (np. .private, .common)
    if target.name =~ /^(callstack-repack|SwiftyRSA|JWTDecode)(\.|$)/
      target.build_configurations.each do |config|
        config.build_settings['BUILD_LIBRARY_FOR_DISTRIBUTION'] = 'NO'
      end
    end

    target.build_configurations.each do |config|
      config.build_settings['ONLY_ACTIVE_ARCH'] = 'YES'
    end
  end
end
