  installer.aggregate_targets.each do |aggregate_target|
    next unless aggregate_target.name == 'Bridge'

    aggregate_target.user_targets.each do |user_target|
      user_target.shell_script_build_phases
        .select { |phase| phase.name == '[CP] Copy Pods Resources' }
        .each { |phase| user_target.build_phases.delete(phase) }
    end
  end
