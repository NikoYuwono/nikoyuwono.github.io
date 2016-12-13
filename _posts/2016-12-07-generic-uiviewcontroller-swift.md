---
layout: post
title: Generic UIViewController in Swift
description: Using Generic UIViewController we can eliminate a lot of duplicate code and produce cleaner code. Even with the disadvantage of cannot using Interface Builder, I think it still worth to use Generic UIViewController especially when your view is simple but used in multiple location. 
---

## Motivation

It's not a rare case for us to have two or more similar screen in our apps. If the data type is same, it won't be a big problem we can create an UIViewController that will adapt based on the usage. But what if the data type is different for each screen?  
We have these choices :

#### - Create one UIViewController that will adapt based on data types 
This woud be the easiest way to implement, but you will end up with a viewcontroller that contain multiple data types and a lot of UI customization code inside it that will result messy and unreadable code.

#### - Create multiple UIViewController based on data type
This will be better solution than the first one because it will separate the UI customization code for each data types and make sure each UIViewController will only have one data type. With this solution, we can get simpler and cleaner UIViewControllers that are more readable compared to previous one but you will also get a lot of duplicate code because they have similar view. 

#### - Create one Generic UIViewController and create additional UIViewController if needed (UI Customization)
Using generic, we can solve the previous problem with duplicate code but still keeping it flexible with possibility of UI customization for each data types in their own UIViewController. Let's see how we can implement this in next Section.


## Generic UIViewController

Like I wrote above, we can use Generic UIViewController to create flexible and reusable UIViewController that can work with any type, subject to requirements that you define. First, let's see the code if we use the second solution.

FoodSelectionViewController.swift

```  
class FoodSelectionViewController: UITableViewController {
    
    var foods: [Food]?
    var selectedFood: Food?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        self.tableView?.tintColor = UIColor.foodCellColor()
    }
    
    override func tableView(tableView: UITableView, heightForRowAtIndexPath indexPath: NSIndexPath) -> CGFloat {
        return UITableViewAutomaticDimension
    }
    
    override func tableView(tableView: UITableView, didSelectRowAtIndexPath indexPath: NSIndexPath) {
        guard let food = self.foods?[indexPath.row] else { return }
      	self.selectedFood = food
    }
    
    override func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.foods?.count ?? 0
    }
    
    override func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
        // Cell creation code for FoodSelectionViewController
    }
}
```

DrinkSelectionViewController.swift

``` 
class DrinkSelectionViewController: UITableViewController {
    
    var drinks: [Drink]?
    var selectedDrink: Drink?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        self.tableView?.tintColor = UIColor.drinkCellColor()
    }
    
    override func tableView(tableView: UITableView, heightForRowAtIndexPath indexPath: NSIndexPath) -> CGFloat {
        return UITableViewAutomaticDimension
    }
    
    override func tableView(tableView: UITableView, didSelectRowAtIndexPath indexPath: NSIndexPath) {
        guard let drink = self.drinks?[indexPath.row] else { return }
      	self.selectedDrink = drink
    }
    
    override func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.drinks?.count ?? 0
    }
    
    override func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
        // Cell creation code for DrinkSelectionViewController
    }
}
```

As you can see FoodSelectionViewController and DrinkSelectionViewController looks really similar. Let's see how we can eliminate the duplicate code with Generic.

SelectionViewController.swift

``` 
class ItemSelectionViewController<T: Item>: UITableViewController {

    var items: [T]?
    var selectedItem: T?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        self.tableView = UITableView(frame: UIScreen.mainScreen().bounds, style: .Plain)
        self.tableView.delegate = self
        self.tableView.dataSource = self
        self.tableView.registerNib(UINib(nibName: "ItemSelectionCell", bundle: nil), forCellReuseIdentifier: "ItemSelectionCell")
        self.tableView.backgroundColor = UIColor.whiteColor()
    }
    
    override func tableView(tableView: UITableView, heightForRowAtIndexPath indexPath: NSIndexPath) -> CGFloat {
        return UITableViewAutomaticDimension
    }
    
    override func tableView(tableView: UITableView, didSelectRowAtIndexPath indexPath: NSIndexPath) {
        guard let item = self.items?[indexPath.row] else { return }
      	self.selectedItem = item
    }
    
    override func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.items?.count ?? 0
    }
}
``` 

FoodSelectionViewController.swift

```  
class FoodSelectionViewController: ItemSelectionViewController<Food> {
    
    override func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
        // Cell customization code for FoodSelectionViewController
    }
}
```

DrinkSelectionViewController.swift

``` 
class DrinkSelectionViewController: ItemSelectionViewController<Drink> {
    
    override func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
        // Cell customization code for DrinkSelectionViewController
    }
}
```

Using Generic, we need to declare the UIViewController like this :

```
class ItemSelectionViewController<T: Item>: UITableViewController
```

The part inside matching angle brackets (`<>`) is called Type Parameters. You need to specify what Type you want to be passed into that class. In this case I want ItemSelectionViewController to only accept Type that is instance of Item.  
Afer we specify the TypeParameter, the compiler now can know what Type that needed to be used inside that class. In our case that would be items and selectedItem. For example, in FoodSelectionViewController we passed Food as Type Parameter to ItemSelectionViewController

```
class FoodSelectionViewController: ItemSelectionViewController<Food>
```

then items and selectedItem inside FoodSelectionViewController will become 

```
var items: [Food]?
var selectedItem: Food?
```

## Disadvantage of Generic UIViewController

The disadvantage of using Generic UIViewController is we can't use InterfaceBuilder or Storyboard to be associated with the UIViewController. It because Interface Builder still use Objective-C runtime to communicate with our code (See [this](http://stackoverflow.com/questions/25263882/use-a-generic-class-as-a-custom-view-in-interface-builder) discussion for more reference).   
So the current solution is to create the view programatically until apple change the InterfaceBuilder to run on Swift.

## Bonus : Generic in Protocol

AS you can see above the sample I wrote is an Item Selection Screen, usually we will have a delegate that will inform the previous screen which item has been selected. Instead of declaring multiple protocol for each data types, we can create one Protocol with Associated Types. An associated type gives a placeholder name to a type that is used as part of the protocol. The actual type to use for that associated type is not specified until the protocol is adopted.

Let's see how can we implement that in our code :

SelectionViewController.swift

``` 
protocol ItemSelectionViewControllerDelegate: class {
    associatedtype T
    func didSelect(item: T)
}

class ItemSelectionViewController<T: Item, D: ItemSelectionViewControllerDelegate where D.T == T>: UITableViewController {

    var items: [T]?
    var selectedItem: T?
    weak var delegate: D?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        self.tableView = UITableView(frame: UIScreen.mainScreen().bounds, style: .Plain)
        self.tableView.delegate = self
        self.tableView.dataSource = self
        self.tableView.registerNib(UINib(nibName: "ItemSelectionCell", bundle: nil), forCellReuseIdentifier: "ItemSelectionCell")
        self.tableView.backgroundColor = UIColor.whiteColor()
    }
    
    override func tableView(tableView: UITableView, heightForRowAtIndexPath indexPath: NSIndexPath) -> CGFloat {
        return UITableViewAutomaticDimension
    }
    
    override func tableView(tableView: UITableView, didSelectRowAtIndexPath indexPath: NSIndexPath) {
        guard let item = self.items?[indexPath.row], 
            let delegate = delegate 
            else { return }
      	
        delegate.didSelect(abuseReason)
        self.navigationController?.popViewControllerAnimated(true)
    }
    
    override func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.items?.count ?? 0
    }
}
``` 

FoodOrderViewController.swift

```
class FoodOrderViewController: UIViewController, ItemSelectionViewControllerDelegate {

    typealias T = Food
    
    func didSelect(item: T) {
        self.selectedFood = item
    }
}
```
FoodSelectionViewController.swift

```  
class FoodSelectionViewController: ItemSelectionViewController<Food, FoodOrderViewController> {
    
    override func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
        // Cell customization code for FoodSelectionViewController
    }
}
```

So now we have two Type Parameters in ItemSelectionViewController

```
class ItemSelectionViewController<T: Item, D: ItemSelectionViewControllerDelegate where D.T == T>: UITableViewController
```

One is item type that will determine the type of `var items: [T]?` and `var selectedItem: T?`. Second one is delegate class that have declared the associated type equal to the item type (`T`).  
With declaring it like this we don't need to declare multiple protocol for each data type, but only one protocol that can handle different data types.

## Ending Words

Using Generic UIViewController we can eliminate a lot of duplicate code and produce cleaner code. Even with the disadvantage of cannot using Interface Builder, I think it still worth to use Generic UIViewController especially when your view is simple but used in multiple location.  
Thanks for reading and if you have any question you can write comment below or reach me via twitter [@niko_yuwono](https://twitter.com/niko_yuwono).
