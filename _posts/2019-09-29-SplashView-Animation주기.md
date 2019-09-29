---
published: false
---

# Splash에 Animation 효과 주기 & CAAnimationDelegate  

정적인 앱 로딩 화면에 마치 LaunchScreen이 움직이는 것과 같은 효과를 주기 위해서는 MainController에 몇가지 작업이 필요합니다. 그리고 Animation 효과를 주기 위해 `CAAnimationDelegate`에 관해 사용방법을 기록하겠습니다.  

<br>

<img width="561" alt="image" src="https://user-images.githubusercontent.com/33486820/65831558-f2ce9300-e2f5-11e9-8c62-3ec670afcc4a.png">

우선 위와 같이 Twitter앱의 효과(실제 트위터앱과는 관련이 없습니다!)를 주기위해 LauchScreen의 스토리보드에 위와 같이 asset을 설정한다. 여기서 주의할 점은 저 twitter asset의 사이즈를 잘 기억하고 있어야한 현재 asset의 size는 `width: 100, height: 80`이다.  


<br>

그런다음 LaunchScreen 스토리보드의 역할은 끝이났다. 모든 작업은 초기 진입이 될 ViewController에서 작업을 하면된다.  

## Splash Animation 구현 코드  

```swift
import UIKit

class ViewController: UIViewController {
    
    @IBOutlet weak var imageView: UIImageView!
    
    var mask: CALayer?
    
    override var prefersStatusBarHidden: Bool {
        return true
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        setUpCALayerMask()
        animateMask()
    }
    
    func setUpCALayerMask() {
        self.mask = CALayer()
        self.mask!.contents = UIImage(named: "twitter")?.cgImage
        self.mask?.contentsGravity = CALayerContentsGravity.resizeAspect
        self.mask?.bounds = CGRect(x: 0, y: 0, width: 100, height: 81)
        self.mask?.anchorPoint = CGPoint(x: 0.5, y: 0.5)
        self.imageView.setNeedsLayout()
        self.imageView.layoutIfNeeded()
        self.mask?.position = CGPoint(x: self.imageView.frame.size.width / 2 , y: self.imageView.frame.height / 2)
        self.imageView.layer.mask = mask
    }
    
    func animateMask() {
        let keyFrameAnimation = CAKeyframeAnimation(keyPath: "bounds")
        keyFrameAnimation.delegate = self
        keyFrameAnimation.duration = 0.6
        keyFrameAnimation.beginTime = CACurrentMediaTime() + 0.5
        keyFrameAnimation.timingFunctions = [CAMediaTimingFunction(name: .easeInEaseOut), CAMediaTimingFunction(name: .easeInEaseOut)]
        
        do {
            // kCAFillModeForwards 에서 변경 -> CAMediaTimingFillMode.forwards
            keyFrameAnimation.fillMode = CAMediaTimingFillMode.forwards
            keyFrameAnimation.isRemovedOnCompletion = false
        }
        
        let initalBounds = NSValue(cgRect: mask!.bounds)
        let secondBounds = NSValue(cgRect: CGRect(x: 0, y: 0, width: 90*0.9, height: 73*0.9))
        let finalBounds = NSValue(cgRect: CGRect(x: 0, y: 0, width: 1600, height: 1300))
        
        keyFrameAnimation.values = [initalBounds, secondBounds, finalBounds]
        keyFrameAnimation.keyTimes = [0, 0.3, 1]
        self.mask?.add(keyFrameAnimation, forKey: "bounds")
        
    }

}

extension ViewController: CAAnimationDelegate {
    func animationDidStop(_ anim: CAAnimation, finished flag: Bool) {
        self.imageView.layer.mask = nil
    }
}
```
