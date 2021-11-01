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

## 리펙토링한 코드
```swift
class MultiSelectFilterDefaultTableViewCell: UITableViewCell { 
    var filterButton: UIButton?

    func setFilterButton(with item: String?) {
        filterButton.setTitle(item, for: .normal)
    }
}
class MultiSelectFilterDescriptionTableViewCell: UITableViewCell { 
    var filterButton: UIButton?
    var descriptionLabel: UILabel?

    func setFilterButton(with item: String?) {
        filterButton.setTitle(item, for: .normal)
    }

    func setDescriptionLabel(with description: String?) {
        descriptionLabel.isHidden = description.isEmpty
        descriptionLabel.text = description
    }
}
class MultiSelectFilterDisableTableViewCell: UITableViewCell { }

extension UITableViewCell {
    static var className: String {
        return String(describing: Self.self)
    }
    static var nibName: String {
        return self.className
    }
    static var identifier: String {
        return self.className
    }
}

private func registerMultiSelectTableViewCells() {
    tableView.register(UINib(nibName: MultiSelectFilterDefaultTableViewCell.nibName, 
                             bundle: nil), 
                       forCellReuseIdentifier: MultiSelectFilterDefaultTableViewCell.identifier)
    tableView.register(UINib(nibName: MultiSelectFilterDescriptionTableViewCell.nibName, 
                             bundle: nil), 
                       forCellReuseIdentifier: MultiSelectFilterDescriptionTableViewCell.identifier)
    tableView.register(UINib(nibName: MultiSelectFilterDisableTableViewCell.nibName, 
                             bundle: nil), 
                       forCellReuseIdentifier: MultiSelectFilterDisableTableViewCell.identifier)
}
     
registerMultiSelectTableViewCells()

list.bind(to: tableView.rx.items) { [weak self] (tableView, row, item) -> UITableViewCell in
    let currentIndexPath = IndexPath(row: row, section: 0)
    if self?.type == .Age,
       !(0...2 ~= row) {
        let cell = tableView.dequeueReusableCell(withIdentifier: MultiSelectFilterDescriptionTableViewCell.identifier, 
                                                 for: currentIndexPath) as! MultiSelectFilterDescriptionTableViewCell
        let description = self?.viewModel.getAgeDescription(row: row) ?? ""                                       
	    cell.setFilterButton(with: item)
	    cell.setDescriptionLabel(with: description)
        return cell
    }
    else {
        let cell = tableView.dequeueReusableCell(withIdentifier: MultiSelectFilterDefaultTableViewCell.identifier, 
                                                 for: currentIndexPath) as! MultiSelectFilterDefaultTableViewCell
        cell.setFilterButton(with: item)
        return cell
    }
}.disposed(by: disposeBag)
```

## 코드 설명

### 코드내에 상수를 관련 타입으로 빼주었습니다.

- 모든 TableViewCell 들이 자신의 클래스 이름을 nib 이름과 identifier로 가지고 있기 때문에 extension 을 관련 변수를 넣어주었습니다.

```swift
extension UITableViewCell {
    static var className: String {
        return String(describing: Self.self)
    }
    static var nibName: String {
        return self.className
    }
    static var identifier: String {
        return self.className
    }
}

tableView.register(UINib(nibName: MultiSelectFilterDefaultTableViewCell.nibName, 
                         bundle: nil), 
	           forCellReuseIdentifier: MultiSelectFilterDefaultTableViewCell.identifier)
```

### 여러개의 cell을 등록하는 함수를 한개의 함수로 묶어주었습니다.

```swift
private func registerMultiSelectTableViewCells() {
    tableView.register(UINib(nibName: MultiSelectFilterDefaultTableViewCell.nibName, 
                             bundle: nil), 
                       forCellReuseIdentifier: MultiSelectFilterDefaultTableViewCell.identifier)
    tableView.register(UINib(nibName: MultiSelectFilterDescriptionTableViewCell.nibName, 
                             bundle: nil), 
                       forCellReuseIdentifier: MultiSelectFilterDescriptionTableViewCell.identifier)
    tableView.register(UINib(nibName: MultiSelectFilterDisableTableViewCell.nibName, 
                             bundle: nil), 
                       forCellReuseIdentifier: MultiSelectFilterDisableTableViewCell.identifier)
}

registerMultiSelectTableViewCells()
```

### cell 내부의 UI들을 설정하는 것을 cell 함수로 넣어주었습니다.

- viewController 함수에서 cell의 프로퍼티를 접근하게 되면 두 인스턴스 간에 결합도 강해집니다. 결합도를 줄이기 위해 수정했습니다.

```swift
var filterButton: UIButton?
var descriptionLabel: UILabel?

func setFilterButton(with item: String?) {
    filterButton.setTitle(item, for: .normal)
}

func setDescriptionLabel(with description: String?) {
    descriptionLabel.isHidden = description.isEmpty
    descriptionLabel.text = description
}
```

### 이외 중복되는 코드들은 중복을 줄여주었습니다.

- indexPath를 하나의 변수에 할당하고 사용했습니다.

```swift
let currentIndexPath = IndexPath(row: row, section: 0)
```

- `MultiSelectFilterDescriptionTableViewCell` 를 사용하는 경우를 제외하고는 나머지 경우를 else로 처리해주었습니다.

```swift
if self?.type == .Age,
   !(0...2 ~= row) {
    let cell = tableView.dequeueReusableCell(withIdentifier: MultiSelectFilterDescriptionTableViewCell.identifier, 
                                             for: currentIndexPath) as! MultiSelectFilterDescriptionTableViewCell
    let description = self?.viewModel.getAgeDescription(row: row) ?? ""                                       
    cell.setFilterButton(with: item)
    cell.setDescriptionLabel(with: description)
    return cell
}
else {
    let cell = tableView.dequeueReusableCell(withIdentifier: MultiSelectFilterDefaultTableViewCell.identifier, 
                                             for: currentIndexPath) as! MultiSelectFilterDefaultTableViewCell
    cell.setFilterButton(with: item)
    return cell
}
```
