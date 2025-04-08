post_install do |installer|
  react_native_post_install(installer, config[:reactNativePath])

  is_bridge_target = installer.aggregate_targets.any? { |t| t.name.downcase.include?('bridge') }

  installer.pods_project.targets.each do |target|
    if ['callstack-repack', 'SwiftyRSA', 'JWTDecode'].include?(target.name)
      target.build_configurations.each do |config|
        config.build_settings['BUILD_LIBRARY_FOR_DISTRIBUTION'] = 'NO'
      end
    end

    target.build_configurations.each do |config|
      config.build_settings['ONLY_ACTIVE_ARCH'] = 'YES'
    end

    # ✅ Usuń tylko bundle z privacy targetu — ale tylko w Bridge
    if is_bridge_target && target.name.end_with?('_privacy')
      target.build_phases.each do |phase|
        next unless phase.respond_to?(:files)
        phase.files.each do |file|
          if file.display_name.end_with?('.xcprivacy') || file.display_name.end_with?('.bundle') || file.display_name.include?('privacy')
            phase.remove_file_reference(file.file_ref)
          end
        end
      end
    end
  end
end
