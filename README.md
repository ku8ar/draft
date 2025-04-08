ENV['RCT_NEW_ARCH_ENABLED'] = '0'

require Pod::Executable.execute_command('node', ['-p',
  "require.resolve(
    'react-native/scripts/react_native_pods.rb',
    {paths: [process.argv[1]]}
  )", __dir__]).strip

platform :ios, min_ios_version_supported
prepare_react_native_project!

linkage = ENV['USE_FRAMEWORKS']
if linkage
  Pod::UI.puts "Using #{linkage}ally linked frameworks"
  use_frameworks! :linkage => linkage.to_sym
end

config = use_native_modules!

target 'WCSOnboarding' do
  use_react_native!(
    :path => config[:reactNativePath],
    :app_path => "#{Pod::Config.instance.installation_root}/.."
  )

  pod 'JWTDecode', '3.0.1'

  target 'WCSOnboardingTests' do
    inherit! :complete
  end
end

target 'Bridge' do
  use_react_native!(
    :path => config[:reactNativePath],
    :app_path => "#{Pod::Config.instance.installation_root}/.."
  )
end

# ✅ Statyczna kompilacja bibliotek Swift, które nie wspierają swiftinterface
pre_install do |installer|
  installer.pod_targets.each do |pod|
    if ['SwiftyRSA', 'callstack-repack', 'JWTDecode'].include?(pod.name)
      def pod.build_type; Pod::BuildType.static_library; end
    end
  end
end

# ✅ Usunięcie privacy targetów i .private tylko dla Bridge
post_install do |installer|
  react_native_post_install(installer, config[:reactNativePath])

  is_bridge = installer.aggregate_targets.any? { |t| t.name.downcase.include?('bridge') }

  installer.pods_project.targets.each do |target|
    if is_bridge && (target.name.end_with?('_privacy') || target.name.end_with?('.private'))
      target.remove_from_project
      next
    end

    target.build_configurations.each do |config|
      config.build_settings['ONLY_ACTIVE_ARCH'] = 'YES'

      # BUILD_LIBRARY_FOR_DISTRIBUTION = YES tylko w Bridge
      if is_bridge && target.name == 'Bridge'
        config.build_settings['BUILD_LIBRARY_FOR_DISTRIBUTION'] = 'YES'
      else
        config.build_settings['BUILD_LIBRARY_FOR_DISTRIBUTION'] = 'NO'
      end
    end
  end
end
