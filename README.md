vc.view.translatesAutoresizingMaskIntoConstraints = false
view.addSubview(vc.view)

NSLayoutConstraint.activate([
    vc.view.topAnchor.constraint(equalTo: view.topAnchor),
    vc.view.bottomAnchor.constraint(equalTo: view.bottomAnchor),
    vc.view.leadingAnchor.constraint(equalTo: view.leadingAnchor),
    vc.view.trailingAnchor.constraint(equalTo: view.trailingAnchor)
])
