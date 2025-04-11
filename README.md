  app_project_path = File.join(__dir__, 'Bridge.xcodeproj')
  app_project = Xcodeproj::Project.open(app_project_path)

  # 🔍 znajdź target 'Bridge'
  bridge_target = app_project.targets.find { |t| t.name == 'Bridge' }

  if bridge_target
    bridge_target.shell_script_build_phases.each do |phase|
      if phase.name == '[CP] Copy Pods Resources'
        # 🧼 Nadpisz zawartość skryptu, by nic nie robił
        phase.shell_script = 'echo "disabled [CP] Copy Pods Resources for Bridge"'
      end
    end

    # 💾 Zapisz zmiany
    app_project.save
  end
