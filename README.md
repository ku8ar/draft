post_install do |installer|
  ...
  updateSandboxSyncMessagesIfNeeded()
end

def updateSandboxSyncMessagesIfNeeded
  updateSandboxSyncMessageIfNeeded('MyApplication.xcodeproj')
end

def updateSandboxSyncMessageIfNeeded(projectFile)
  project = Xcodeproj::Project.open(projectFile)
  changed = false

  project.targets.each do |target|
    target.build_phases.each do |build_phase|
      if defined?(build_phase.shell_script) && build_phase.shell_script.include?("pod install")
        script = build_phase.shell_script.gsub("The sandbox is not in sync with the Podfile.lock. Run 'pod install' or update your CocoaPods installation.",
                                               "The sandbox is not in sync, please run './my-fancy-script.sh' to fix.")
        build_phase.shell_script = script
        changed = true
      end
    end
  end

  if changed
    project.save()
  end
end
