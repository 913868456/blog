
---
title:  TextView自适应高度 
date:  2019-01-11 18:23
categories:
- iOS
tags: 
- UITextView 
---
#### 应用场景
 TableView 的Cell内的textView根据文本输入内容,cell的高度自适应
#### 实现方式
- TableView Cell高度自适应 
```
        editList.estimatedRowHeight = 52;  //设置预估行高
        editList.rowHeight = UITableViewAutomaticDimension;  //设置Cell高度自适应
```
-  Cell内部设置
```
      var textString: String {
        get {
            return textView?.text ?? ""
        }
        set {
            if let textView = textView {
                textView.text = newValue
                textViewDidChange(textView) //在cell外部给textView赋值,需要调用textView的代理方法,重新计算textView的size
            }
        }
      }
     func setupUI() {
        
        selectionStyle = .none;
        separatorInset = UIEdgeInsetsMake(0, 20, 0, 20);
        contentView.backgroundColor = .white;
        titleLabel = UILabel();
        titleLabel.textAlignment = .left;
        titleLabel.textColor = ResourceManager.LIGHT_TEXT_COLOR;
        titleLabel.font = ResourceManager.NORMAL_FONT;
        contentView.addSubview(titleLabel);
        
        textView = UITextView();
        textView.textAlignment = .right;
        textView.delegate = self;
        textView.isScrollEnabled = false; //禁止textView的滚动
        textView.font = ResourceManager.NORMAL_FONT;
       
        contentView.addSubview(textView);
        
        titleLabel.setContentCompressionResistancePriority(.defaultHigh, for: .horizontal);
        textView.setContentCompressionResistancePriority(.defaultLow, for: .horizontal);
        
    }
    
    override func layoutSubviews() {
        super.layoutSubviews();
        titleLabel.snp.makeConstraints { (make) in
            make.top.equalToSuperview().offset(16);
            make.leading.equalToSuperview().offset(20);
        }
        
        textView.snp.makeConstraints { (make) in
            make.top.equalToSuperview().offset(8);
            make.bottom.equalToSuperview().offset(-8);
            make.trailing.equalToSuperview().offset(-20);
            make.leading.equalTo(titleLabel.snp.trailing).offset(8);
        }
    }

        //textView代理方法
        func textViewDidChange(_ textView: UITextView) {
              let size = textView.bounds.size
              let newSize = textView.sizeThatFits(CGSize(width: size.width, height: CGFloat.greatestFiniteMagnitude))
        
            if let controller = self.parentViewController() as? ContractEditController {
            // Resize the cell only when cell's size is changed
            if size.height != newSize.height {
                UIView.setAnimationsEnabled(false)
                controller.editList.beginUpdates()
                controller.editList.endUpdates()
                UIView.setAnimationsEnabled(true)
                
                if let thisIndexPath = controller.editList.indexPath(for: self) {//滚动到cell底部
                    controller.editList.scrollToRow(at: thisIndexPath, at: .bottom, animated: false)
                }
              } 
            }
         }

```
