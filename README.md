post_install do |installer|
  installer.pods_project.targets.each do |target|
    if target.name == 'Bridge'
      target.shell_script_build_phases
            .select { |phase| phase.name == '[CP] Copy Pods Resources' }
            .each { |phase| target.build_phases.delete(phase) }
    end
  end
end
