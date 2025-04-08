  installer.aggregate_targets.each do |aggregate_target|
    if aggregate_target.name == 'Bridge'
      aggregate_target.user_project.native_targets.each do |t|
        if t.name == 'Bridge'
          t.build_configurations.each do |config|
            config.build_settings['BUILD_LIBRARY_FOR_DISTRIBUTION'] = 'YES'
          end
        end
      end
    end
  end
