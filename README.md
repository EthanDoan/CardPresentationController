# CardPresentationController

Custom [UIPresentationController](https://developer.apple.com/documentation/uikit/uipresentationcontroller) which mimics the behavior of Apple Music UI.

[DEMO video on iPhone Xs simulator](CardPresentationController.mp4)

## Installation
### Manually

Add the folder `CardPresentationController` into your project. It's only five files.

### CocoaPods

Don't support now, later make it work because I'm still working on some larger features(see TODO section at the end). 

### Carthage

Don't support now, later make it work because I'm still working on some larger features(see TODO section at the end). 




## Usage

From anywhere you want to present some `UIViewController`, call

```swift
let vc = ...
presentCard(vc, animated: true)
```

You dismiss it as any other modal:

```swift
dismiss(animated: true)
```

This will present `vc` modally, flying-in from the bottom edge. Existing view will be kept shown as dimmed background card, on black background.

You can *present card from another card*; library will stack the cards nicely. Do use common sense as popups over popups don’t make pleasant user experience.

### Advanced behavior

If the _presenting_ controller was `UINavigationController` instance (which is the case in most apps) then its `barStyle` will be automatically changed to `.black`. So it looks dimmed.

If `presenting` is not `UINavigationController` instance, then its view will be (by default) 20% transparent to blend into the background a bit, again looking dimmed.

In both cases that back "card" is inset a bit from the edges.

![](resources/presentedNC-top.png)

If the _presented_ VC is `UINavigationController` instance, nothing special happens. It’s assumed that you will add `UIBarButtonItem` which will facilitate dismissal.

If it is not, then `CardPresentationController` will automatically add a button at the middle of the shown card. Tapping on that will dismiss the cards.

![](resources/presentedVC-top.png)

As you present card over card, back cards will be ever more transparent and horizontally inset. In most cases, this should look rather nice.

### Status bar style

CardPresentationController tries its best to enforce `.lightContent` status bar style. You can help it, by adding this into your UIVC subclass:

```swift
override var preferredStatusBarStyle: UIStatusBarStyle {
	return .lightContent
}
```

If you are presenting UINC, then my advice is to subclass it and override `preferredStatusBarStyle` property in the same way.

## Requirements

Currently it’s tested only on iOS. 

It *requires iOS 10*, since it uses [UIViewPropertyAnimator](https://developer.apple.com/documentation/uikit/uiviewpropertyanimator), [UISpringTimingParameters](https://developer.apple.com/documentation/uikit/uispringtimingparameters) and a bunch of other modern UIKit animation APIs.

On iOS 11 it uses [maskedCorners](https://developer.apple.com/documentation/quartzcore/calayer/2877488-maskedcorners) property to round just the top corners. On iOS 10.x it will fallback to rounding all corners.

## How it works

The main object here is `CardTransitionManager`, which acts as  `UIViewControllerTransitioningDelegate`. It is internally instantiated and assigned as property on UIVC which called `presentCard()` – that's _sourceController_ in the UIPresentationController parlance.

This instance of CTM is automatically removed on dismissal.

CTM creates and manages the other two required objects:

* `CardPresentationController`: manages additional views (like dismiss handle at the top of the card) and other aspects of the custom presentation
* `CardAnimator`: which performs the animated transition

In case you missed it — *you don’t deal with any of that*. It’s all implementation detail, hidden inside these 3 classes. You never instantiate them directly.

The only object you can put to use, if you want to, is…

### CardConfiguration

When calling `presentCard`, you can supply optional `CardConfiguration` instance. This is simple struct containing the following parameters:

```swift
///	Vertical inset from the top or already shown card
var verticalSpacing: CGFloat = 16

///	Leading and trailing inset for the existing (presenting) view 
/// when it's being pushed further back
var horizontalInset: CGFloat = 16

///	Cards have rounded corners, right?
var cornerRadius: CGFloat = 12

///	The starting frame for the presented card.
var initialTransitionFrame: CGRect?

///	How much to fade the back card.
///
///	Ignored if back card is UINavigationController.
var backFadeAlpha: CGFloat = 0.8
```

There’s a very handy `init` for it where you can supply any combination of these parameters.

### Advanced example

Thus if you want to control where the card originates — say if you want to mimic Apple Music's now-playing card — you can:

```swift
let vc = ContentController.instantiate()

let f = container.convert(sender.bounds, to: view.window!)
let config = CardConfiguration(initialTransitionFrame: f)

presentCard(vc,
			configuration: config,
			animated: true)
```

The important bit here is setting `initialTransitionFrame` property to the frame *in the UIWindow coordinating space*, since transition happens in it.

### Caveats

`CardAnimator` animates layout of its own subviews – `from` and `to` views included in `transitionContext`. Behavior and layout of the internal subviews of both _presented_ and _presenting_/_source_ views is up to you *but* CardAnimator will try its best to animate them along.

Depending on the complexity of your UI, in may be impossible to make the transition perfect. Usually in cases where UIKit applies its own private API magic related to status / navigation bars. 
See `EmbeddedNCExample` where I have `UINavigationController` embedded inside ordinary `UIViewController`. This is very unusual UIVC stack which I would love to solve since I have project using just that.

## TODO

* ~~configurability~~ (1.2)
* ~~stack multiple cards~~ (1.1)
* interactivity (both for presenting and dismissing)
* improve landscape behavior
* ~~more examples~~ (1.1)

## LICENSE

[MIT](LICENSE), as usual for all my stuff.


