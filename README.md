      if target.name.downcase.include?('swiftyrsa') || target.name.downcase.include?('jwtdecode') || target.name.downcase.include?('callstack')
        config.build_settings['BUILD_LIBRARY_FOR_DISTRIBUTION'] = 'NO'
      end
