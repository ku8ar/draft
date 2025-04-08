ENV['RCT_NEW_ARCH_ENABLED'] = '0'

# Resolve react_native_pods.rb with node to allow for hoisting
require Pod::Executable.execute_command('node', ['-p',
  "require.resolve(
    'react-native/scripts/react_native_pods.rb',
    {paths: [process.argv[1]]}
  )", __dir__]).strip

platform :ios, min_ios_version_supported
prepare_react_native_project!

linkage = ENV['USE_FRAMEWORKS']
if linkage != nil
  Pod::UI.puts "Configuring Pod with #{linkage}ally linked Frameworks".green
  use_frameworks! :linkage => linkage.to_sym
end

config = use_native_modules!

target 'WCSOnboarding' do
  use_react_native!(
    :path => config[:reactNativePath],
    :app_path => "#{Pod::Config.instance.installation_root}/.."
  )

  pod 'SwiftyRSA'
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

# ✅ Wymuś statyczne budowanie dla Swiftowych podów
pre_install do |installer|
  installer.pod_targets.each do |pod|
    if ['SwiftyRSA', 'callstack-repack', 'JWTDecode'].include?(pod.name)
      def pod.build_type; Pod::BuildType.static_library; end
    end
  end
end

# ✅ Usuń *_privacy tylko w Bridge i wyłącz BUILD_LIBRARY_FOR_DISTRIBUTION tam gdzie trzeba
post_install do |installer|
  react_native_post_install(installer, config[:reactNativePath])

  is_bridge_target = installer.aggregate_targets.any? { |t| t.name.downcase.include?('bridge') }

  installer.pods_project.targets.each do |target|
    # 🔥 Usuń privacy bundle tylko w Bridge
    if is_bridge_target && target.name.end_with?('_privacy')
      target.remove_from_project
      next
    end

    # 🔧 Wyłącz BUILD_LIBRARY_FOR_DISTRIBUTION dla Swift podów (i ich odmian)
    if target.name.match?(/^(callstack-repack|SwiftyRSA|JWTDecode)(\.|$)/)
      target.build_configurations.each do |config|
        config.build_settings['BUILD_LIBRARY_FOR_DISTRIBUTION'] = 'NO'
      end
    end

    target.build_configurations.each do |config|
      config.build_settings['ONLY_ACTIVE_ARCH'] = 'YES'
    end
  end
end
