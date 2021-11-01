## 문제
클린코드 적용 문제
아래의 내용은 째깍악어의 코드중 일부를 발췌한 내용입니다. 해당부분을 클린코드를 적용해주세요. (봇사용 금지)
```swift
    tableView.register(UINib(nibName: "MultiSelectFilterDefaultTableViewCell", bundle: nil), forCellReuseIdentifier: "MultiSelectFilterDefaultTableViewCell")
    tableView.register(UINib(nibName: "MultiSelectFilterDescriptionTableViewCell", bundle: nil), forCellReuseIdentifier: "MultiSelectFilterDescriptionTableViewCell")
    tableView.register(UINib(nibName: "MultiSelectFilterDisableTableViewCell", bundle: nil), forCellReuseIdentifier: "MultiSelectFilterDisableTableViewCell")
     
    list.bind(to: tableView.rx.items) { [weak self] (tableView, row, item) -> UITableViewCell in
      switch self?.type {
      case .CareType:
        let cell = tableView.dequeueReusableCell(withIdentifier: "MultiSelectFilterDefaultTableViewCell", for: IndexPath(row: row, section: 0)) as! MultiSelectFilterDefaultTableViewCell
        cell.filterButton.setTitle(item, for: .normal)
        return cell
      case .Age:
        if 0...2 ~= row {
          let cell = tableView.dequeueReusableCell(withIdentifier: "MultiSelectFilterDefaultTableViewCell", for: IndexPath(row: row, section: 0)) as! MultiSelectFilterDefaultTableViewCell
          cell.filterButton.setTitle(item, for: .normal)
          return cell
        }
        let cell = tableView.dequeueReusableCell(withIdentifier: "MultiSelectFilterDescriptionTableViewCell", for: IndexPath(row: row, section: 0)) as! MultiSelectFilterDescriptionTableViewCell
        cell.filterButton.setTitle(item, for: .normal)
        let description = self?.viewModel.getAgeDescription(row: row) ?? ""
        cell.descriptionLabel.isHidden = description.isEmpty
        cell.descriptionLabel.text = description
        return cell
      default:
        let cell = tableView.dequeueReusableCell(withIdentifier: "MultiSelectFilterDefaultTableViewCell", for: IndexPath(row: row, section: 0)) as! MultiSelectFilterDefaultTableViewCell
        cell.filterButton.setTitle(item, for: .normal)
        return cell
      }
    }.disposed(by: disposeBag)
```
