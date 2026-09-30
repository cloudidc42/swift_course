# Part 64: Core Graphics และ Drawing

## สารบัญ

1. [Core Graphics Overview](#core-graphics-overview)
2. [CGContext](#cgcontext)
3. [Drawing Shapes](#drawing-shapes)
4. [CGPath และ UIBezierPath](#cgpath-และ-uibezierpath)
5. [Filling และ Stroking](#filling-และ-stroking)
6. [Colors](#colors)
7. [Gradients](#gradients)
8. [Shadows](#shadows)
9. [Transformations](#transformations)
10. [Images ใน Core Graphics](#images-ใน-core-graphics)
11. [PDF Generation](#pdf-generation)
12. [Drawing Text ด้วย Core Text](#drawing-text-ด้วย-core-text)
13. [Custom UIView ด้วย draw(_:)](#custom-uiview-ด้วย-draw)
14. [CALayer](#calayer)
15. [CAShapeLayer](#cashapelayer)
16. [CAGradientLayer](#cagradientlayer)
17. [CATextLayer](#catextlayer)
18. [CAAnimationGroup](#caanimationgroup)
19. [Core Animation Basics](#core-animation-basics)
20. [SwiftUI Canvas vs Core Graphics](#swiftui-canvas-vs-core-graphics)
21. [Metal สำหรับ Custom Rendering](#metal-สำหรับ-custom-rendering)
22. [SpriteKit Intro](#spritekit-intro)
23. [Practical Exercises](#practical-exercises)
24. [Building a Custom Chart App](#building-a-custom-chart-app)
25. [สรุป](#สรุป)

---

## Core Graphics Overview

**Core Graphics** (หรือที่รู้จักในชื่อ **Quartz 2D**) เป็น low-level, 2D rendering engine ของ Apple ที่รองรับทั้ง iOS, macOS, watchOS และ tvOS

### สถาปัตยกรรมของ Core Graphics

```
App Code (Swift/Objective-C)
          ↓
    Core Graphics (Quartz 2D)
          ↓
    Core Animation
          ↓
    OpenGL ES / Metal
          ↓
    GPU Hardware
```

### ความสามารถหลักของ Core Graphics

- **Path-based drawing**: วาด shapes ด้วย mathematical paths
- **Anti-aliased rendering**: ขอบที่เรียบสวยงาม
- **Color spaces**: รองรับ RGB, CMYK, Grayscale, Pattern
- **Gradients**: Linear และ Radial gradients
- **PDF generation**: สร้างและแสดงผล PDF
- **Image manipulation**: Transform, blend, crop images
- **Pattern drawing**: ลายซ้ำๆ
- **Transparency**: Alpha blending

### Coordinate System

```
iOS: Origin ที่ top-left, Y axis ลงล่าง
macOS: Origin ที่ bottom-left, Y axis ขึ้นบน

iOS Coordinates:
(0,0) ────────────→ X+
  │
  │
  ↓
  Y+

macOS Coordinates:
  Y+
  ↑
  │
  │
(0,0) ────────────→ X+
```

---

## CGContext

**CGContext** คือ "canvas" ที่เราใช้วาดทุกอย่างใน Core Graphics

### สร้าง CGContext

```swift
import CoreGraphics
import UIKit

// สร้าง bitmap context สำหรับ off-screen rendering
func createBitmapContext(size: CGSize) -> CGContext? {
    let colorSpace = CGColorSpaceCreateDeviceRGB()
    let bytesPerPixel = 4
    let bytesPerRow = bytesPerPixel * Int(size.width)
    let bitsPerComponent = 8
    
    let bitmapInfo = CGBitmapInfo(rawValue: CGImageAlphaInfo.premultipliedLast.rawValue)
    
    return CGContext(
        data: nil,
        width: Int(size.width),
        height: Int(size.height),
        bitsPerComponent: bitsPerComponent,
        bytesPerRow: bytesPerRow,
        space: colorSpace,
        bitmapInfo: bitmapInfo.rawValue
    )
}

// สร้าง context ผ่าน UIGraphicsBeginImageContext
func drawWithUIKit() -> UIImage? {
    let size = CGSize(width: 200, height: 200)
    
    UIGraphicsBeginImageContextWithOptions(size, false, 0.0)
    defer { UIGraphicsEndImageContext() }
    
    guard let context = UIGraphicsGetCurrentContext() else { return nil }
    
    // วาดบน context
    context.setFillColor(UIColor.blue.cgColor)
    context.fill(CGRect(x: 0, y: 0, width: size.width, height: size.height))
    
    return UIGraphicsGetImageFromCurrentImageContext()
}

// Swift 5.9+ ใช้ UIGraphicsImageRenderer (แนะนำ)
func drawWithRenderer() -> UIImage {
    let renderer = UIGraphicsImageRenderer(size: CGSize(width: 200, height: 200))
    
    return renderer.image { context in
        let cgContext = context.cgContext
        
        // วาดพื้นหลัง
        UIColor.systemBlue.setFill()
        UIRectFill(CGRect(x: 0, y: 0, width: 200, height: 200))
        
        // วาด circle
        UIColor.white.setFill()
        let circle = UIBezierPath(ovalIn: CGRect(x: 50, y: 50, width: 100, height: 100))
        circle.fill()
    }
}
```

### State Machine ของ CGContext

CGContext ทำงานเป็น state machine:

```swift
class ContextStateExample {
    func demonstrateContextState() {
        let renderer = UIGraphicsImageRenderer(size: CGSize(width: 300, height: 300))
        
        let _ = renderer.image { context in
            let ctx = context.cgContext
            
            // State 1: Default state
            ctx.setFillColor(UIColor.red.cgColor)
            ctx.fill(CGRect(x: 0, y: 0, width: 100, height: 100))
            
            // บันทึก state ปัจจุบัน
            ctx.saveGState()
            
            // State 2: Modified state
            ctx.setFillColor(UIColor.blue.cgColor)
            ctx.setAlpha(0.5)
            ctx.translateBy(x: 100, y: 100)
            ctx.fill(CGRect(x: 0, y: 0, width: 100, height: 100))
            
            // คืน state เดิม
            ctx.restoreGState()
            
            // กลับมา State 1: สีแดง, alpha เต็ม
            ctx.fill(CGRect(x: 200, y: 0, width: 100, height: 100))
        }
    }
}
```

### Context Properties ที่สำคัญ

```swift
extension CGContext {
    func setupDrawingContext() {
        // สีขีด (stroke)
        setStrokeColor(UIColor.black.cgColor)
        
        // สีเติม (fill)
        setFillColor(UIColor.blue.cgColor)
        
        // ความหนาของเส้น
        setLineWidth(2.0)
        
        // ความโปร่งใส
        setAlpha(0.8)
        
        // Line cap style
        setLineCap(.round)    // .butt, .round, .square
        
        // Line join style
        setLineJoin(.round)   // .miter, .round, .bevel
        
        // Dash pattern
        setLineDash(phase: 0, lengths: [5, 3])
        
        // Blend mode
        setBlendMode(.normal)  // .multiply, .screen, .overlay, etc.
        
        // Anti-aliasing
        setShouldAntialias(true)
    }
}
```

---

## Drawing Shapes

### วาด Rectangles

```swift
import UIKit

class RectangleDrawing: UIView {
    
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext() else { return }
        
        // วาด filled rectangle
        context.setFillColor(UIColor.systemBlue.cgColor)
        context.fill(CGRect(x: 20, y: 20, width: 100, height: 80))
        
        // วาด stroked rectangle
        context.setStrokeColor(UIColor.systemRed.cgColor)
        context.setLineWidth(3.0)
        context.stroke(CGRect(x: 140, y: 20, width: 100, height: 80))
        
        // วาด rounded rectangle
        let roundedRect = CGRect(x: 20, y: 120, width: 100, height: 80)
        let cornerRadius: CGFloat = 15
        
        let path = UIBezierPath(
            roundedRect: roundedRect,
            cornerRadius: cornerRadius
        )
        
        UIColor.systemGreen.setFill()
        path.fill()
        
        UIColor.black.setStroke()
        path.lineWidth = 2
        path.stroke()
    }
}
```

### วาด Circles และ Ellipses

```swift
class CircleDrawing: UIView {
    
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext() else { return }
        
        // วาด filled circle
        let circleRect = CGRect(x: 50, y: 50, width: 100, height: 100)
        context.setFillColor(UIColor.systemPurple.cgColor)
        context.fillEllipse(in: circleRect)
        
        // วาด stroked ellipse
        let ellipseRect = CGRect(x: 170, y: 50, width: 150, height: 100)
        context.setStrokeColor(UIColor.systemOrange.cgColor)
        context.setLineWidth(3.0)
        context.strokeEllipse(in: ellipseRect)
        
        // วาด arc
        let center = CGPoint(x: 150, y: 250)
        let radius: CGFloat = 80
        let startAngle: CGFloat = 0
        let endAngle: CGFloat = .pi * 1.5  // 270 degrees
        
        context.setFillColor(UIColor.systemYellow.cgColor)
        context.move(to: center)
        context.addArc(
            center: center,
            radius: radius,
            startAngle: startAngle,
            endAngle: endAngle,
            clockwise: false
        )
        context.closePath()
        context.fillPath()
    }
}
```

### วาด Lines

```swift
class LineDrawing: UIView {
    
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext() else { return }
        
        // เส้นตรง
        context.setStrokeColor(UIColor.black.cgColor)
        context.setLineWidth(2.0)
        context.move(to: CGPoint(x: 20, y: 20))
        context.addLine(to: CGPoint(x: 200, y: 20))
        context.strokePath()
        
        // เส้น dashed
        context.setLineDash(phase: 0, lengths: [8, 4])
        context.move(to: CGPoint(x: 20, y: 60))
        context.addLine(to: CGPoint(x: 200, y: 60))
        context.strokePath()
        
        // reset dash
        context.setLineDash(phase: 0, lengths: [])
        
        // เส้น dotted
        context.setLineDash(phase: 0, lengths: [1, 5])
        context.setLineCap(.round)
        context.move(to: CGPoint(x: 20, y: 100))
        context.addLine(to: CGPoint(x: 200, y: 100))
        context.strokePath()
        
        // เส้นโค้ง (Bezier curve)
        context.setLineDash(phase: 0, lengths: [])
        context.setStrokeColor(UIColor.systemBlue.cgColor)
        context.setLineWidth(3.0)
        context.move(to: CGPoint(x: 20, y: 200))
        context.addCurve(
            to: CGPoint(x: 300, y: 200),
            control1: CGPoint(x: 100, y: 100),
            control2: CGPoint(x: 200, y: 300)
        )
        context.strokePath()
    }
}
```

---

## CGPath และ UIBezierPath

### UIBezierPath

**UIBezierPath** เป็น high-level wrapper รอบ CGPath ที่ใช้ง่ายกว่า:

```swift
import UIKit

class BezierPathExamples {
    
    // สร้าง star shape
    func starPath(center: CGPoint, radius: CGFloat, points: Int) -> UIBezierPath {
        let path = UIBezierPath()
        let innerRadius = radius * 0.4
        let angle = -CGFloat.pi / 2  // เริ่มที่บน
        let step = CGFloat.pi * 2 / CGFloat(points)
        
        for i in 0..<points * 2 {
            let r = i % 2 == 0 ? radius : innerRadius
            let currentAngle = angle + CGFloat(i) * step / 2
            let point = CGPoint(
                x: center.x + r * cos(currentAngle),
                y: center.y + r * sin(currentAngle)
            )
            
            if i == 0 {
                path.move(to: point)
            } else {
                path.addLine(to: point)
            }
        }
        
        path.close()
        return path
    }
    
    // สร้าง arrow shape
    func arrowPath(from start: CGPoint, to end: CGPoint, width: CGFloat) -> UIBezierPath {
        let path = UIBezierPath()
        
        let angle = atan2(end.y - start.y, end.x - start.x)
        let arrowLength = sqrt(pow(end.x - start.x, 2) + pow(end.y - start.y, 2))
        let headLength = min(width * 3, arrowLength * 0.3)
        let shaftLength = arrowLength - headLength
        
        path.move(to: CGPoint(
            x: start.x + (width / 2) * sin(angle),
            y: start.y - (width / 2) * cos(angle)
        ))
        path.addLine(to: CGPoint(
            x: start.x + shaftLength * cos(angle) + (width / 2) * sin(angle),
            y: start.y + shaftLength * sin(angle) - (width / 2) * cos(angle)
        ))
        path.addLine(to: CGPoint(
            x: start.x + shaftLength * cos(angle) + width * sin(angle),
            y: start.y + shaftLength * sin(angle) - width * cos(angle)
        ))
        path.addLine(to: end)
        path.addLine(to: CGPoint(
            x: start.x + shaftLength * cos(angle) - width * sin(angle),
            y: start.y + shaftLength * sin(angle) + width * cos(angle)
        ))
        path.addLine(to: CGPoint(
            x: start.x + shaftLength * cos(angle) - (width / 2) * sin(angle),
            y: start.y + shaftLength * sin(angle) + (width / 2) * cos(angle)
        ))
        path.addLine(to: CGPoint(
            x: start.x - (width / 2) * sin(angle),
            y: start.y + (width / 2) * cos(angle)
        ))
        path.close()
        
        return path
    }
    
    // สร้าง speech bubble
    func speechBubblePath(rect: CGRect, tailSize: CGFloat = 20) -> UIBezierPath {
        let radius: CGFloat = 12
        let path = UIBezierPath()
        
        let x = rect.minX
        let y = rect.minY
        let w = rect.width
        let h = rect.height - tailSize
        
        // Top-left corner
        path.move(to: CGPoint(x: x + radius, y: y))
        
        // Top edge
        path.addLine(to: CGPoint(x: x + w - radius, y: y))
        
        // Top-right corner
        path.addArc(
            withCenter: CGPoint(x: x + w - radius, y: y + radius),
            radius: radius,
            startAngle: -.pi / 2,
            endAngle: 0,
            clockwise: true
        )
        
        // Right edge
        path.addLine(to: CGPoint(x: x + w, y: y + h - radius))
        
        // Bottom-right corner
        path.addArc(
            withCenter: CGPoint(x: x + w - radius, y: y + h - radius),
            radius: radius,
            startAngle: 0,
            endAngle: .pi / 2,
            clockwise: true
        )
        
        // Bottom edge with tail
        path.addLine(to: CGPoint(x: x + w / 2 + tailSize / 2, y: y + h))
        
        // Tail
        path.addLine(to: CGPoint(x: x + w / 2, y: y + h + tailSize))
        path.addLine(to: CGPoint(x: x + w / 2 - tailSize / 2, y: y + h))
        
        path.addLine(to: CGPoint(x: x + radius, y: y + h))
        
        // Bottom-left corner
        path.addArc(
            withCenter: CGPoint(x: x + radius, y: y + h - radius),
            radius: radius,
            startAngle: .pi / 2,
            endAngle: .pi,
            clockwise: true
        )
        
        // Left edge
        path.addLine(to: CGPoint(x: x, y: y + radius))
        
        // Top-left corner
        path.addArc(
            withCenter: CGPoint(x: x + radius, y: y + radius),
            radius: radius,
            startAngle: .pi,
            endAngle: -.pi / 2,
            clockwise: true
        )
        
        path.close()
        return path
    }
}
```

### CGPath

```swift
import CoreGraphics

class CGPathExamples {
    
    // CGPath ที่ mutable
    func createMutablePath() -> CGPath {
        let path = CGMutablePath()
        
        path.move(to: CGPoint(x: 100, y: 100))
        path.addLine(to: CGPoint(x: 200, y: 100))
        path.addLine(to: CGPoint(x: 200, y: 200))
        path.closeSubpath()
        
        return path
    }
    
    // Path operations
    func pathOperations() {
        let path1 = CGMutablePath()
        path1.addRect(CGRect(x: 0, y: 0, width: 100, height: 100))
        
        let path2 = CGMutablePath()
        path2.addEllipse(in: CGRect(x: 25, y: 25, width: 50, height: 50))
        
        // ตรวจสอบว่า point อยู่ใน path หรือไม่
        let testPoint = CGPoint(x: 50, y: 50)
        let isInside = path1.contains(testPoint)
        print("Point is inside: \(isInside)")  // true
        
        // Bounding box
        let boundingBox = path1.boundingBox
        print("Bounding box: \(boundingBox)")
        
        // แปลง path ด้วย transform
        var transform = CGAffineTransform(scaleX: 2, y: 2)
        let scaledPath = path1.copy(using: &transform)!
        _ = scaledPath.boundingBox  // ใช้ bounding box
    }
}
```

---

## Filling และ Stroking

### Fill Rules

```swift
class FillRuleExamples: UIView {
    
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext() else { return }
        
        // Even-Odd Fill Rule
        let path1 = UIBezierPath()
        // วงนอก
        path1.append(UIBezierPath(ovalIn: CGRect(x: 20, y: 20, width: 120, height: 120)))
        // วงใน (จะเป็น hole)
        path1.append(UIBezierPath(ovalIn: CGRect(x: 50, y: 50, width: 60, height: 60)))
        
        path1.usesEvenOddFillRule = true
        UIColor.systemBlue.setFill()
        path1.fill()
        
        // Winding Number Fill Rule
        let path2 = UIBezierPath()
        path2.append(UIBezierPath(ovalIn: CGRect(x: 170, y: 20, width: 120, height: 120)))
        path2.append(UIBezierPath(ovalIn: CGRect(x: 200, y: 50, width: 60, height: 60)))
        
        path2.usesEvenOddFillRule = false  // Non-zero (default)
        UIColor.systemRed.setFill()
        path2.fill()
    }
}
```

### Clipping

```swift
class ClippingExample: UIView {
    
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext() else { return }
        
        // บันทึก state
        context.saveGState()
        
        // สร้าง clip path เป็น circle
        let clipPath = UIBezierPath(
            ovalIn: CGRect(x: 50, y: 50, width: 200, height: 200)
        )
        clipPath.addClip()
        
        // วาด gradient ภายใน clip region
        let colors = [UIColor.systemBlue.cgColor, UIColor.systemPurple.cgColor] as CFArray
        let colorSpace = CGColorSpaceCreateDeviceRGB()
        let gradient = CGGradient(
            colorsSpace: colorSpace,
            colors: colors,
            locations: [0.0, 1.0]
        )!
        
        context.drawLinearGradient(
            gradient,
            start: CGPoint(x: 50, y: 50),
            end: CGPoint(x: 250, y: 250),
            options: []
        )
        
        // คืน state (remove clip)
        context.restoreGState()
        
        // วาดนอก clip region ปกติ
        context.setFillColor(UIColor.systemRed.withAlphaComponent(0.5).cgColor)
        context.fill(CGRect(x: 0, y: 0, width: bounds.width, height: bounds.height))
    }
}
```

---

## Colors

### CGColor vs UIColor

```swift
import UIKit
import CoreGraphics

class ColorExamples {
    
    // UIColor
    func uiColorExamples() {
        let red = UIColor.red
        let custom = UIColor(red: 0.2, green: 0.5, blue: 0.8, alpha: 1.0)
        let hex = UIColor(
            red: CGFloat(0x4A) / 255.0,
            green: CGFloat(0x90) / 255.0,
            blue: CGFloat(0xD9) / 255.0,
            alpha: 1.0
        )
        
        // Dynamic colors (Dark Mode support)
        let dynamicColor = UIColor { traitCollection in
            if traitCollection.userInterfaceStyle == .dark {
                return UIColor.white
            } else {
                return UIColor.black
            }
        }
        
        _ = (red, custom, hex, dynamicColor)  // suppress unused warnings
    }
    
    // CGColor
    func cgColorExamples() {
        let colorSpace = CGColorSpaceCreateDeviceRGB()
        
        // สร้างจาก components
        let components: [CGFloat] = [1.0, 0.0, 0.0, 1.0]  // R, G, B, A
        let cgRed = CGColor(colorSpace: colorSpace, components: components)!
        
        // แปลงจาก UIColor
        let cgBlue = UIColor.blue.cgColor
        
        // ดึง components
        if let comps = cgBlue.components {
            print("R: \(comps[0]), G: \(comps[1]), B: \(comps[2]), A: \(comps[3])")
        }
        
        _ = cgRed
    }
    
    // สร้าง UIColor extension สำหรับ hex
    func hexColor(_ hex: String) -> UIColor {
        var hexString = hex.trimmingCharacters(in: .whitespacesAndNewlines)
        hexString = hexString.hasPrefix("#") ? String(hexString.dropFirst()) : hexString
        
        guard hexString.count == 6 else { return .black }
        
        var rgb: UInt64 = 0
        Scanner(string: hexString).scanHexInt64(&rgb)
        
        return UIColor(
            red: CGFloat((rgb & 0xFF0000) >> 16) / 255.0,
            green: CGFloat((rgb & 0x00FF00) >> 8) / 255.0,
            blue: CGFloat(rgb & 0x0000FF) / 255.0,
            alpha: 1.0
        )
    }
}
```

---

## Gradients

### Linear Gradient

```swift
import UIKit

class GradientExamples: UIView {
    
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext() else { return }
        
        drawLinearGradient(context: context, in: rect)
        drawRadialGradient(context: context)
    }
    
    func drawLinearGradient(context: CGContext, in rect: CGRect) {
        let colorSpace = CGColorSpaceCreateDeviceRGB()
        
        let colors = [
            UIColor(red: 0.98, green: 0.36, blue: 0.35, alpha: 1.0).cgColor,
            UIColor(red: 0.98, green: 0.72, blue: 0.35, alpha: 1.0).cgColor,
            UIColor(red: 0.36, green: 0.73, blue: 0.98, alpha: 1.0).cgColor
        ] as CFArray
        
        let locations: [CGFloat] = [0.0, 0.5, 1.0]
        
        guard let gradient = CGGradient(
            colorsSpace: colorSpace,
            colors: colors,
            locations: locations
        ) else { return }
        
        // Horizontal gradient
        context.saveGState()
        context.addRect(CGRect(x: 20, y: 20, width: bounds.width - 40, height: 80))
        context.clip()
        
        context.drawLinearGradient(
            gradient,
            start: CGPoint(x: 20, y: 60),
            end: CGPoint(x: bounds.width - 20, y: 60),
            options: [.drawsBeforeStartLocation, .drawsAfterEndLocation]
        )
        context.restoreGState()
    }
    
    func drawRadialGradient(context: CGContext) {
        let colorSpace = CGColorSpaceCreateDeviceRGB()
        
        let colors = [
            UIColor(white: 1.0, alpha: 1.0).cgColor,
            UIColor(white: 0.0, alpha: 1.0).cgColor
        ] as CFArray
        
        guard let gradient = CGGradient(
            colorsSpace: colorSpace,
            colors: colors,
            locations: nil  // nil = equal spacing
        ) else { return }
        
        let center = CGPoint(x: bounds.width / 2, y: 200)
        
        context.drawRadialGradient(
            gradient,
            startCenter: center,
            startRadius: 0,
            endCenter: center,
            endRadius: 100,
            options: [.drawsBeforeStartLocation, .drawsAfterEndLocation]
        )
    }
}
```

### CAGradientLayer (Layer-based Gradient)

```swift
import UIKit

class GradientView: UIView {
    
    let gradientLayer = CAGradientLayer()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupGradient()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        setupGradient()
    }
    
    private func setupGradient() {
        gradientLayer.colors = [
            UIColor.systemBlue.cgColor,
            UIColor.systemPurple.cgColor
        ]
        
        // จุดเริ่มต้นและสิ้นสุด (0.0 ถึง 1.0 ใน unit coordinate space)
        gradientLayer.startPoint = CGPoint(x: 0, y: 0)      // top-left
        gradientLayer.endPoint = CGPoint(x: 1, y: 1)        // bottom-right
        
        // Locations (optional)
        gradientLayer.locations = [0.0, 1.0]
        
        layer.addSublayer(gradientLayer)
    }
    
    override func layoutSubviews() {
        super.layoutSubviews()
        gradientLayer.frame = bounds
    }
    
    // Animate gradient
    func animateGradient(to newColors: [UIColor]) {
        let animation = CABasicAnimation(keyPath: "colors")
        animation.fromValue = gradientLayer.colors
        animation.toValue = newColors.map { $0.cgColor }
        animation.duration = 1.0
        animation.fillMode = .forwards
        animation.isRemovedOnCompletion = false
        
        gradientLayer.add(animation, forKey: "colorChange")
        gradientLayer.colors = newColors.map { $0.cgColor }
    }
}
```

---

## Shadows

```swift
class ShadowExamples: UIView {
    
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext() else { return }
        
        // Shadow ด้วย CGContext
        context.saveGState()
        
        // กำหนด shadow
        let shadowColor = UIColor.black.withAlphaComponent(0.3).cgColor
        context.setShadow(
            offset: CGSize(width: 3, height: 3),
            blur: 8,
            color: shadowColor
        )
        
        // วาด shape (shadow จะถูก apply อัตโนมัติ)
        context.setFillColor(UIColor.white.cgColor)
        let path = UIBezierPath(
            roundedRect: CGRect(x: 40, y: 40, width: 200, height: 120),
            cornerRadius: 16
        )
        context.addPath(path.cgPath)
        context.fillPath()
        
        context.restoreGState()
        
        // Shadow กับ transparency layer
        context.saveGState()
        
        context.setShadow(
            offset: CGSize(width: 5, height: 5),
            blur: 10,
            color: UIColor.systemBlue.withAlphaComponent(0.4).cgColor
        )
        
        // เริ่ม transparency layer สำหรับ shadow ที่ถูกต้องกับ complex shapes
        context.beginTransparencyLayer(auxiliaryInfo: nil)
        
        context.setFillColor(UIColor.systemPurple.cgColor)
        context.fillEllipse(in: CGRect(x: 60, y: 200, width: 100, height: 100))
        
        context.setFillColor(UIColor.systemPink.cgColor)
        context.fillEllipse(in: CGRect(x: 120, y: 220, width: 100, height: 100))
        
        context.endTransparencyLayer()
        
        context.restoreGState()
    }
}
```

---

## Transformations

```swift
class TransformationExamples: UIView {
    
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext() else { return }
        
        let squareSize = CGRect(x: 0, y: 0, width: 60, height: 60)
        
        // Translation (เลื่อน)
        context.saveGState()
        context.translateBy(x: 50, y: 50)
        context.setFillColor(UIColor.systemRed.cgColor)
        context.fill(squareSize)
        context.restoreGState()
        
        // Rotation (หมุน)
        context.saveGState()
        context.translateBy(x: 200, y: 80)  // เลื่อนไปก่อน pivot point
        context.rotate(by: .pi / 4)          // 45 degrees
        context.translateBy(x: -30, y: -30)  // offset กลับ
        context.setFillColor(UIColor.systemBlue.cgColor)
        context.fill(squareSize)
        context.restoreGState()
        
        // Scale (ย่อ/ขยาย)
        context.saveGState()
        context.translateBy(x: 50, y: 200)
        context.scaleBy(x: 1.5, y: 0.8)
        context.setFillColor(UIColor.systemGreen.cgColor)
        context.fill(squareSize)
        context.restoreGState()
        
        // CGAffineTransform
        let transform = CGAffineTransform(translationX: 200, y: 200)
            .rotated(by: .pi / 6)
            .scaledBy(x: 1.2, y: 1.2)
        
        context.saveGState()
        context.concatenate(transform)
        context.setFillColor(UIColor.systemOrange.cgColor)
        context.fill(squareSize)
        context.restoreGState()
    }
}
```

### UIBezierPath Transform

```swift
class PathTransformExample {
    
    func transformedPath() {
        let originalPath = UIBezierPath(
            rect: CGRect(x: 0, y: 0, width: 100, height: 100)
        )
        
        // Apply transform ไปที่ path
        let rotationTransform = CGAffineTransform(rotationAngle: .pi / 4)
        originalPath.apply(rotationTransform)
        
        // สร้าง path ใหม่จาก transform
        let scaledPath = UIBezierPath(cgPath: originalPath.cgPath)
        let scaleTransform = CGAffineTransform(scaleX: 2, y: 2)
        scaledPath.apply(scaleTransform)
    }
}
```

---

## Images ใน Core Graphics

```swift
class ImageDrawing: UIView {
    
    var image: UIImage?
    
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext(),
              let cgImage = image?.cgImage else { return }
        
        let imageRect = CGRect(x: 20, y: 20, width: 200, height: 200)
        
        // วาด image (สังเกต: Core Graphics coordinate เป็น flipped)
        context.saveGState()
        context.translateBy(x: 0, y: bounds.height)  // flip coordinate
        context.scaleBy(x: 1, y: -1)
        
        // คำนวณ rect ใน flipped coordinate
        let flippedRect = CGRect(
            x: imageRect.minX,
            y: bounds.height - imageRect.maxY,
            width: imageRect.width,
            height: imageRect.height
        )
        
        context.draw(cgImage, in: flippedRect)
        context.restoreGState()
        
        // วิธีที่ง่ายกว่า: ใช้ UIImage.draw(in:)
        image?.draw(in: CGRect(x: 230, y: 20, width: 100, height: 100))
        
        // Tinted image
        if let tintedImage = image {
            context.saveGState()
            
            // สร้าง mask จาก image
            context.clip(to: CGRect(x: 20, y: 250, width: 100, height: 100),
                        mask: tintedImage.cgImage!)
            
            // เติมสีทับ
            context.setFillColor(UIColor.systemBlue.cgColor)
            context.fill(CGRect(x: 20, y: 250, width: 100, height: 100))
            
            context.restoreGState()
        }
    }
    
    // สร้าง image ด้วย Core Graphics
    static func createCustomImage(size: CGSize) -> UIImage {
        let renderer = UIGraphicsImageRenderer(size: size)
        
        return renderer.image { context in
            // Draw gradient background
            let colors = [UIColor.systemBlue.cgColor, UIColor.systemPurple.cgColor] as CFArray
            let colorSpace = CGColorSpaceCreateDeviceRGB()
            
            if let gradient = CGGradient(
                colorsSpace: colorSpace,
                colors: colors,
                locations: nil
            ) {
                context.cgContext.drawLinearGradient(
                    gradient,
                    start: .zero,
                    end: CGPoint(x: size.width, y: size.height),
                    options: []
                )
            }
            
            // Draw circular icon
            UIColor.white.withAlphaComponent(0.8).setFill()
            let circleRect = CGRect(
                x: size.width / 4,
                y: size.height / 4,
                width: size.width / 2,
                height: size.height / 2
            )
            UIBezierPath(ovalIn: circleRect).fill()
        }
    }
}
```

---

## PDF Generation

```swift
import UIKit

class PDFGenerator {
    
    // สร้าง PDF ง่ายๆ
    func createSimplePDF() -> Data {
        let pdfMetaData = [
            kCGPDFContextCreator: "My App",
            kCGPDFContextAuthor: "Developer",
            kCGPDFContextTitle: "My Report"
        ]
        
        let format = UIGraphicsPDFRendererFormat()
        format.documentInfo = pdfMetaData as [String: Any]
        
        let pageSize = CGSize(width: 595.2, height: 841.8)  // A4
        let renderer = UIGraphicsPDFRenderer(bounds: CGRect(origin: .zero, size: pageSize), format: format)
        
        let data = renderer.pdfData { context in
            // หน้าแรก
            context.beginPage()
            
            // วาด header
            drawHeader(context: context.cgContext, in: CGRect(x: 40, y: 40, width: 515, height: 80))
            
            // วาด content
            drawContent(context: context, pageSize: pageSize)
            
            // หน้าสอง
            context.beginPage()
            
            let attributes: [NSAttributedString.Key: Any] = [
                .font: UIFont.systemFont(ofSize: 14),
                .foregroundColor: UIColor.black
            ]
            
            let text = "หน้า 2 ของเอกสาร"
            text.draw(at: CGPoint(x: 40, y: 40), withAttributes: attributes)
        }
        
        return data
    }
    
    private func drawHeader(context: CGContext, in rect: CGRect) {
        // Background
        context.setFillColor(UIColor.systemBlue.cgColor)
        context.fill(rect)
        
        // Title
        let titleAttributes: [NSAttributedString.Key: Any] = [
            .font: UIFont.boldSystemFont(ofSize: 24),
            .foregroundColor: UIColor.white
        ]
        
        let title = "รายงานประจำเดือน"
        let titleSize = title.size(withAttributes: titleAttributes)
        let titleOrigin = CGPoint(
            x: rect.midX - titleSize.width / 2,
            y: rect.midY - titleSize.height / 2
        )
        
        title.draw(at: titleOrigin, withAttributes: titleAttributes)
    }
    
    private func drawContent(context: UIGraphicsPDFRendererContext, pageSize: CGSize) {
        let data = [
            ("มกราคม", 120.0),
            ("กุมภาพันธ์", 85.0),
            ("มีนาคม", 150.0),
            ("เมษายน", 95.0),
            ("พฤษภาคม", 180.0)
        ]
        
        let chartRect = CGRect(x: 40, y: 150, width: 515, height: 300)
        drawBarChart(context: context.cgContext, data: data, in: chartRect)
    }
    
    private func drawBarChart(
        context: CGContext,
        data: [(String, Double)],
        in rect: CGRect
    ) {
        let maxValue = data.map { $0.1 }.max() ?? 1.0
        let barWidth = rect.width / CGFloat(data.count) - 10
        
        for (index, item) in data.enumerated() {
            let barHeight = CGFloat(item.1 / maxValue) * rect.height
            let x = rect.minX + CGFloat(index) * (barWidth + 10)
            let y = rect.maxY - barHeight
            
            let barRect = CGRect(x: x, y: y, width: barWidth, height: barHeight)
            
            context.setFillColor(UIColor.systemBlue.cgColor)
            context.fill(barRect)
            
            // Label
            let labelAttributes: [NSAttributedString.Key: Any] = [
                .font: UIFont.systemFont(ofSize: 10),
                .foregroundColor: UIColor.black
            ]
            
            item.0.draw(
                at: CGPoint(x: x, y: rect.maxY + 5),
                withAttributes: labelAttributes
            )
        }
    }
    
    // บันทึก PDF
    func savePDF(to url: URL) throws {
        let data = createSimplePDF()
        try data.write(to: url)
    }
}
```

---

## Drawing Text ด้วย Core Text

```swift
import CoreText
import UIKit

class CoreTextDrawing: UIView {
    
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext() else { return }
        
        // Flip coordinate system สำหรับ Core Text
        context.textMatrix = .identity
        context.translateBy(x: 0, y: bounds.size.height)
        context.scaleBy(x: 1.0, y: -1.0)
        
        // สร้าง attributed string
        let text = "สวัสดี Core Text! Hello World! 🌟"
        
        let attributedString = NSMutableAttributedString(string: text)
        
        // Font
        let fontDescriptor = CTFontDescriptorCreateWithAttributes([
            kCTFontNameAttribute: "Helvetica-Bold",
            kCTFontSizeAttribute: 18
        ] as CFDictionary)
        
        let font = CTFontCreateWithFontDescriptor(fontDescriptor, 0, nil)
        
        attributedString.addAttribute(
            .font,
            value: font,
            range: NSRange(location: 0, length: attributedString.length)
        )
        
        // Color
        attributedString.addAttribute(
            .foregroundColor,
            value: UIColor.systemBlue.cgColor,
            range: NSRange(location: 0, length: 7)  // "สวัสดี "
        )
        
        attributedString.addAttribute(
            .foregroundColor,
            value: UIColor.systemRed.cgColor,
            range: NSRange(location: 7, length: 12)  // "Core Text! "
        )
        
        // Paragraph style
        let paragraphStyle = NSMutableParagraphStyle()
        paragraphStyle.alignment = .center
        
        attributedString.addAttribute(
            .paragraphStyle,
            value: paragraphStyle,
            range: NSRange(location: 0, length: attributedString.length)
        )
        
        // สร้าง framesetter
        let framesetter = CTFramesetterCreateWithAttributedString(
            attributedString as CFAttributedString
        )
        
        // กำหนด frame
        let framePath = CGPath(
            rect: CGRect(x: 20, y: 20, width: bounds.width - 40, height: 200),
            transform: nil
        )
        
        let frame = CTFramesetterCreateFrame(
            framesetter,
            CFRangeMake(0, attributedString.length),
            framePath,
            nil
        )
        
        // วาด frame
        CTFrameDraw(frame, context)
    }
}
```

---

## Custom UIView ด้วย draw(_:)

```swift
import UIKit

// Custom Progress Ring View
class ProgressRingView: UIView {
    
    // MARK: - Properties
    
    var progress: CGFloat = 0.7 {
        didSet {
            progress = max(0, min(1, progress))  // clamp 0-1
            setNeedsDisplay()
        }
    }
    
    var ringColor: UIColor = .systemBlue {
        didSet { setNeedsDisplay() }
    }
    
    var trackColor: UIColor = .systemGray5 {
        didSet { setNeedsDisplay() }
    }
    
    var lineWidth: CGFloat = 12 {
        didSet { setNeedsDisplay() }
    }
    
    var showPercentage: Bool = true {
        didSet { setNeedsDisplay() }
    }
    
    // MARK: - Drawing
    
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext() else { return }
        
        let center = CGPoint(x: rect.midX, y: rect.midY)
        let radius = min(rect.width, rect.height) / 2 - lineWidth / 2
        
        // วาด track (background ring)
        context.setStrokeColor(trackColor.cgColor)
        context.setLineWidth(lineWidth)
        context.setLineCap(.round)
        
        context.addArc(
            center: center,
            radius: radius,
            startAngle: 0,
            endAngle: .pi * 2,
            clockwise: false
        )
        context.strokePath()
        
        // วาด progress ring
        let startAngle: CGFloat = -.pi / 2  // เริ่มที่บน
        let endAngle = startAngle + (.pi * 2 * progress)
        
        context.setStrokeColor(ringColor.cgColor)
        context.setLineWidth(lineWidth)
        context.setLineCap(.round)
        
        context.addArc(
            center: center,
            radius: radius,
            startAngle: startAngle,
            endAngle: endAngle,
            clockwise: false
        )
        context.strokePath()
        
        // วาด percentage text
        if showPercentage {
            let percentage = Int(progress * 100)
            let text = "\(percentage)%"
            
            let attributes: [NSAttributedString.Key: Any] = [
                .font: UIFont.boldSystemFont(ofSize: min(rect.width, rect.height) * 0.2),
                .foregroundColor: UIColor.label
            ]
            
            let textSize = text.size(withAttributes: attributes)
            let textOrigin = CGPoint(
                x: center.x - textSize.width / 2,
                y: center.y - textSize.height / 2
            )
            
            text.draw(at: textOrigin, withAttributes: attributes)
        }
    }
    
    // MARK: - Animation
    
    func animate(to newProgress: CGFloat, duration: TimeInterval = 1.0) {
        let animation = CABasicAnimation(keyPath: "progress")
        animation.fromValue = progress
        animation.toValue = newProgress
        animation.duration = duration
        animation.timingFunction = CAMediaTimingFunction(name: .easeInEaseOut)
        
        // ใช้ display link สำหรับ smooth animation
        let displayLink = CADisplayLink(target: self, selector: #selector(updateProgress))
        displayLink.add(to: .current, forMode: .default)
        
        let startTime = CACurrentMediaTime()
        let startProgress = progress
        
        Timer.scheduledTimer(withTimeInterval: 1.0 / 60.0, repeats: true) { [weak self, weak displayLink] timer in
            guard let self = self else {
                timer.invalidate()
                return
            }
            
            let elapsed = CACurrentMediaTime() - startTime
            let fraction = min(elapsed / duration, 1.0)
            
            // Ease in-out
            let easedFraction = fraction < 0.5
                ? 2 * fraction * fraction
                : 1 - pow(-2 * fraction + 2, 2) / 2
            
            self.progress = startProgress + (newProgress - startProgress) * CGFloat(easedFraction)
            
            if fraction >= 1.0 {
                timer.invalidate()
                displayLink?.invalidate()
            }
        }
    }
    
    @objc private func updateProgress() {
        setNeedsDisplay()
    }
}

// การใช้งาน
class ExampleViewController: UIViewController {
    
    let progressRing = ProgressRingView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        progressRing.frame = CGRect(x: 0, y: 0, width: 200, height: 200)
        progressRing.center = view.center
        progressRing.backgroundColor = .clear
        progressRing.progress = 0
        progressRing.ringColor = .systemGreen
        
        view.addSubview(progressRing)
        
        DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
            self.progressRing.animate(to: 0.75, duration: 1.5)
        }
    }
}
```

---

## CALayer

**CALayer** เป็น foundation ของ UIView's visual appearance:

```swift
import UIKit

class LayerExamples: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupLayers()
    }
    
    func setupLayers() {
        // Basic layer
        let basicLayer = CALayer()
        basicLayer.frame = CGRect(x: 20, y: 100, width: 100, height: 100)
        basicLayer.backgroundColor = UIColor.systemBlue.cgColor
        basicLayer.cornerRadius = 12
        basicLayer.shadowColor = UIColor.black.cgColor
        basicLayer.shadowOffset = CGSize(width: 3, height: 3)
        basicLayer.shadowOpacity = 0.3
        basicLayer.shadowRadius = 8
        view.layer.addSublayer(basicLayer)
        
        // Layer กับ border
        let borderLayer = CALayer()
        borderLayer.frame = CGRect(x: 140, y: 100, width: 100, height: 100)
        borderLayer.backgroundColor = UIColor.white.cgColor
        borderLayer.borderColor = UIColor.systemRed.cgColor
        borderLayer.borderWidth = 3
        borderLayer.cornerRadius = 50  // perfect circle
        view.layer.addSublayer(borderLayer)
        
        // Layer กับ contents (image)
        let imageLayer = CALayer()
        imageLayer.frame = CGRect(x: 260, y: 100, width: 100, height: 100)
        imageLayer.contents = UIImage(systemName: "star.fill")?.cgImage
        imageLayer.contentsGravity = .resizeAspect
        imageLayer.backgroundColor = UIColor.systemYellow.cgColor
        imageLayer.cornerRadius = 12
        view.layer.addSublayer(imageLayer)
        
        // Sublayer
        let parentLayer = CALayer()
        parentLayer.frame = CGRect(x: 20, y: 220, width: 200, height: 100)
        parentLayer.backgroundColor = UIColor.systemGray6.cgColor
        parentLayer.masksToBounds = true
        parentLayer.cornerRadius = 12
        view.layer.addSublayer(parentLayer)
        
        let childLayer = CALayer()
        childLayer.frame = CGRect(x: 50, y: -20, width: 100, height: 100)
        childLayer.backgroundColor = UIColor.systemBlue.cgColor
        childLayer.cornerRadius = 50
        parentLayer.addSublayer(childLayer)
    }
}
```

---

## CAShapeLayer

```swift
import UIKit

class ShapeLayerExamples: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        setupHeartLayer()
        setupPieChart()
        setupAnimatedLine()
    }
    
    func setupHeartLayer() {
        let shapeLayer = CAShapeLayer()
        shapeLayer.frame = CGRect(x: 20, y: 100, width: 100, height: 100)
        
        // สร้าง heart path
        let path = UIBezierPath()
        path.move(to: CGPoint(x: 50, y: 30))
        path.addCurve(
            to: CGPoint(x: 50, y: 85),
            controlPoint1: CGPoint(x: 100, y: -10),
            controlPoint2: CGPoint(x: 100, y: 60)
        )
        path.addCurve(
            to: CGPoint(x: 50, y: 30),
            controlPoint1: CGPoint(x: 0, y: 60),
            controlPoint2: CGPoint(x: 0, y: -10)
        )
        
        shapeLayer.path = path.cgPath
        shapeLayer.fillColor = UIColor.systemRed.cgColor
        shapeLayer.strokeColor = UIColor.red.cgColor
        shapeLayer.lineWidth = 2
        
        view.layer.addSublayer(shapeLayer)
    }
    
    func setupPieChart() {
        let data: [(CGFloat, UIColor)] = [
            (0.4, .systemBlue),
            (0.3, .systemGreen),
            (0.2, .systemOrange),
            (0.1, .systemPurple)
        ]
        
        let center = CGPoint(x: 200, y: 180)
        let radius: CGFloat = 70
        var startAngle: CGFloat = -.pi / 2
        
        for (percentage, color) in data {
            let endAngle = startAngle + (.pi * 2 * percentage)
            
            let path = UIBezierPath()
            path.move(to: center)
            path.addArc(
                withCenter: center,
                radius: radius,
                startAngle: startAngle,
                endAngle: endAngle,
                clockwise: true
            )
            path.close()
            
            let slice = CAShapeLayer()
            slice.path = path.cgPath
            slice.fillColor = color.cgColor
            slice.strokeColor = UIColor.white.cgColor
            slice.lineWidth = 2
            
            view.layer.addSublayer(slice)
            
            startAngle = endAngle
        }
    }
    
    func setupAnimatedLine() {
        let path = UIBezierPath()
        path.move(to: CGPoint(x: 20, y: 350))
        
        let points: [CGPoint] = [
            CGPoint(x: 60, y: 300),
            CGPoint(x: 100, y: 380),
            CGPoint(x: 140, y: 280),
            CGPoint(x: 180, y: 360),
            CGPoint(x: 220, y: 310),
            CGPoint(x: 260, y: 340),
            CGPoint(x: 300, y: 290)
        ]
        
        for point in points {
            path.addLine(to: point)
        }
        
        let lineLayer = CAShapeLayer()
        lineLayer.path = path.cgPath
        lineLayer.fillColor = UIColor.clear.cgColor
        lineLayer.strokeColor = UIColor.systemBlue.cgColor
        lineLayer.lineWidth = 3
        lineLayer.lineCap = .round
        lineLayer.lineJoin = .round
        lineLayer.strokeEnd = 0  // เริ่มต้นที่ 0
        
        view.layer.addSublayer(lineLayer)
        
        // Animate drawing
        let animation = CABasicAnimation(keyPath: "strokeEnd")
        animation.fromValue = 0
        animation.toValue = 1
        animation.duration = 2.0
        animation.timingFunction = CAMediaTimingFunction(name: .easeInEaseOut)
        
        lineLayer.add(animation, forKey: "drawLine")
        lineLayer.strokeEnd = 1
    }
}
```

---

## CAGradientLayer

```swift
import UIKit

class GradientLayerExamples: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        setupLinearGradient()
        setupAngularGradient()
        setupAnimatedGradient()
    }
    
    func setupLinearGradient() {
        let gradientLayer = CAGradientLayer()
        gradientLayer.frame = CGRect(x: 20, y: 100, width: 260, height: 100)
        gradientLayer.cornerRadius = 12
        
        gradientLayer.colors = [
            UIColor(red: 0.98, green: 0.36, blue: 0.35, alpha: 1).cgColor,
            UIColor(red: 0.98, green: 0.72, blue: 0.35, alpha: 1).cgColor
        ]
        
        gradientLayer.startPoint = CGPoint(x: 0, y: 0.5)
        gradientLayer.endPoint = CGPoint(x: 1, y: 0.5)
        
        view.layer.addSublayer(gradientLayer)
    }
    
    func setupAngularGradient() {
        let gradientLayer = CAGradientLayer()
        gradientLayer.frame = CGRect(x: 20, y: 220, width: 120, height: 120)
        gradientLayer.cornerRadius = 60
        
        gradientLayer.type = .conic
        gradientLayer.colors = [
            UIColor.systemRed.cgColor,
            UIColor.systemYellow.cgColor,
            UIColor.systemGreen.cgColor,
            UIColor.systemBlue.cgColor,
            UIColor.systemPurple.cgColor,
            UIColor.systemRed.cgColor
        ]
        gradientLayer.startPoint = CGPoint(x: 0.5, y: 0.5)
        gradientLayer.endPoint = CGPoint(x: 0.5, y: 0)
        
        view.layer.addSublayer(gradientLayer)
    }
    
    func setupAnimatedGradient() {
        let gradientLayer = CAGradientLayer()
        gradientLayer.frame = CGRect(x: 160, y: 220, width: 120, height: 120)
        gradientLayer.cornerRadius = 12
        
        gradientLayer.colors = [
            UIColor.systemBlue.cgColor,
            UIColor.systemPurple.cgColor
        ]
        
        gradientLayer.locations = [0.0, 1.0]
        gradientLayer.startPoint = CGPoint(x: 0, y: 0)
        gradientLayer.endPoint = CGPoint(x: 1, y: 1)
        
        view.layer.addSublayer(gradientLayer)
        
        // Animate colors
        let colorAnimation = CABasicAnimation(keyPath: "colors")
        colorAnimation.fromValue = [
            UIColor.systemBlue.cgColor,
            UIColor.systemPurple.cgColor
        ]
        colorAnimation.toValue = [
            UIColor.systemGreen.cgColor,
            UIColor.systemTeal.cgColor
        ]
        colorAnimation.duration = 2.0
        colorAnimation.autoreverses = true
        colorAnimation.repeatCount = .infinity
        
        gradientLayer.add(colorAnimation, forKey: "colorAnimation")
    }
}
```

---

## CATextLayer

```swift
import UIKit

class TextLayerExamples: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupTextLayers()
    }
    
    func setupTextLayers() {
        // Basic text layer
        let textLayer = CATextLayer()
        textLayer.frame = CGRect(x: 20, y: 100, width: 300, height: 40)
        
        textLayer.string = "Hello, CATextLayer!"
        textLayer.font = UIFont.boldSystemFont(ofSize: 18)
        textLayer.fontSize = 18
        textLayer.foregroundColor = UIColor.label.cgColor
        textLayer.alignmentMode = .center
        textLayer.contentsScale = UIScreen.main.scale  // สำคัญสำหรับ retina
        
        view.layer.addSublayer(textLayer)
        
        // Rich text layer
        let richTextLayer = CATextLayer()
        richTextLayer.frame = CGRect(x: 20, y: 160, width: 300, height: 60)
        
        let attributedString = NSMutableAttributedString(string: "Rich Text Example")
        attributedString.addAttributes(
            [.font: UIFont.boldSystemFont(ofSize: 20),
             .foregroundColor: UIColor.systemBlue],
            range: NSRange(location: 0, length: 4)  // "Rich"
        )
        attributedString.addAttributes(
            [.font: UIFont.italicSystemFont(ofSize: 18),
             .foregroundColor: UIColor.systemRed],
            range: NSRange(location: 5, length: 4)  // "Text"
        )
        
        richTextLayer.string = attributedString
        richTextLayer.contentsScale = UIScreen.main.scale
        richTextLayer.isWrapped = true
        
        view.layer.addSublayer(richTextLayer)
        
        // Truncated text
        let truncatedLayer = CATextLayer()
        truncatedLayer.frame = CGRect(x: 20, y: 240, width: 200, height: 30)
        truncatedLayer.string = "Long text that will be truncated at the end"
        truncatedLayer.fontSize = 14
        truncatedLayer.foregroundColor = UIColor.label.cgColor
        truncatedLayer.truncationMode = .end
        truncatedLayer.contentsScale = UIScreen.main.scale
        
        view.layer.addSublayer(truncatedLayer)
    }
}
```

---

## CAAnimationGroup

```swift
import UIKit

class AnimationGroupExamples: UIViewController {
    
    let animatedLayer = CALayer()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        animatedLayer.frame = CGRect(x: 100, y: 200, width: 80, height: 80)
        animatedLayer.backgroundColor = UIColor.systemBlue.cgColor
        animatedLayer.cornerRadius = 40
        view.layer.addSublayer(animatedLayer)
        
        setupAnimations()
    }
    
    func setupAnimations() {
        // Animation 1: Scale
        let scaleAnimation = CABasicAnimation(keyPath: "transform.scale")
        scaleAnimation.fromValue = 1.0
        scaleAnimation.toValue = 1.5
        scaleAnimation.duration = 1.0
        
        // Animation 2: Color
        let colorAnimation = CABasicAnimation(keyPath: "backgroundColor")
        colorAnimation.fromValue = UIColor.systemBlue.cgColor
        colorAnimation.toValue = UIColor.systemRed.cgColor
        colorAnimation.duration = 1.0
        
        // Animation 3: Position
        let positionAnimation = CAKeyframeAnimation(keyPath: "position")
        positionAnimation.values = [
            CGPoint(x: 140, y: 240),
            CGPoint(x: 240, y: 180),
            CGPoint(x: 240, y: 300),
            CGPoint(x: 140, y: 240)
        ]
        positionAnimation.duration = 2.0
        positionAnimation.timingFunction = CAMediaTimingFunction(name: .easeInEaseOut)
        
        // รวม animations เป็น group
        let group = CAAnimationGroup()
        group.animations = [scaleAnimation, colorAnimation, positionAnimation]
        group.duration = 2.0
        group.autoreverses = true
        group.repeatCount = .infinity
        group.timingFunction = CAMediaTimingFunction(name: .easeInEaseOut)
        
        animatedLayer.add(group, forKey: "animationGroup")
    }
    
    // Pulse animation
    func addPulseAnimation(to layer: CALayer) {
        let pulseGroup = CAAnimationGroup()
        
        let scaleAnim = CABasicAnimation(keyPath: "transform.scale")
        scaleAnim.fromValue = 1.0
        scaleAnim.toValue = 1.2
        
        let opacityAnim = CABasicAnimation(keyPath: "opacity")
        opacityAnim.fromValue = 1.0
        opacityAnim.toValue = 0.6
        
        pulseGroup.animations = [scaleAnim, opacityAnim]
        pulseGroup.duration = 0.8
        pulseGroup.autoreverses = true
        pulseGroup.repeatCount = .infinity
        
        layer.add(pulseGroup, forKey: "pulse")
    }
}
```

---

## Core Animation Basics

```swift
import UIKit

class CoreAnimationExamples: UIViewController {
    
    let square = UIView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        square.frame = CGRect(x: 50, y: 200, width: 80, height: 80)
        square.backgroundColor = .systemBlue
        square.layer.cornerRadius = 12
        view.addSubview(square)
    }
    
    // CABasicAnimation
    func animateWithBasic() {
        let animation = CABasicAnimation(keyPath: "position.x")
        animation.fromValue = square.layer.position.x
        animation.toValue = view.bounds.width - 90
        animation.duration = 0.8
        animation.timingFunction = CAMediaTimingFunction(name: .easeInEaseOut)
        
        // อัปเดต actual position (ไม่งั้น layer จะกลับที่เดิมหลัง animation)
        square.layer.position.x = view.bounds.width - 90
        
        square.layer.add(animation, forKey: "moveRight")
    }
    
    // CAKeyframeAnimation
    func animateWithKeyframes() {
        let animation = CAKeyframeAnimation(keyPath: "position")
        animation.values = [
            CGPoint(x: 90, y: 240),
            CGPoint(x: 200, y: 180),
            CGPoint(x: 300, y: 240),
            CGPoint(x: 200, y: 300),
            CGPoint(x: 90, y: 240)
        ]
        animation.keyTimes = [0, 0.25, 0.5, 0.75, 1.0]
        animation.duration = 2.0
        animation.calculationMode = .catmullRom  // smooth path
        
        square.layer.add(animation, forKey: "orbit")
    }
    
    // CASpringAnimation
    func animateWithSpring() {
        let animation = CASpringAnimation(keyPath: "position.y")
        animation.fromValue = 100
        animation.toValue = 400
        animation.mass = 1.0
        animation.stiffness = 100
        animation.damping = 10
        animation.duration = animation.settlingDuration
        
        square.layer.add(animation, forKey: "spring")
        square.layer.position.y = 400
    }
    
    // UIView Animation (wrapper รอบ CA)
    func animateWithUIView() {
        UIView.animate(
            withDuration: 0.5,
            delay: 0,
            usingSpringWithDamping: 0.7,
            initialSpringVelocity: 0.5,
            options: [.curveEaseInOut],
            animations: {
                self.square.transform = CGAffineTransform(scaleX: 1.5, y: 1.5)
                self.square.alpha = 0.5
                self.square.backgroundColor = .systemRed
            },
            completion: { _ in
                UIView.animate(withDuration: 0.3) {
                    self.square.transform = .identity
                    self.square.alpha = 1.0
                    self.square.backgroundColor = .systemBlue
                }
            }
        )
    }
    
    // Transition animations
    func animateTransition() {
        let transition = CATransition()
        transition.type = .push
        transition.subtype = .fromRight
        transition.duration = 0.4
        transition.timingFunction = CAMediaTimingFunction(name: .easeInEaseOut)
        
        view.layer.add(transition, forKey: kCATransition)
        
        // เปลี่ยน content หลัง transition
        square.backgroundColor = .systemPurple
    }
}
```

---

## SwiftUI Canvas vs Core Graphics

**SwiftUI Canvas** เป็น high-level alternative สำหรับ custom drawing ใน SwiftUI:

```swift
import SwiftUI

// SwiftUI Canvas
struct CanvasExample: View {
    
    @State private var progress: CGFloat = 0.7
    
    var body: some View {
        VStack {
            Canvas { context, size in
                // วาด background
                context.fill(
                    Path(CGRect(origin: .zero, size: size)),
                    with: .color(.systemGray6)
                )
                
                // วาด circles
                for i in 0..<5 {
                    let x = CGFloat(i) * (size.width / 5) + 30
                    let y = size.height / 2
                    let radius: CGFloat = 20
                    
                    var circle = Path()
                    circle.addEllipse(in: CGRect(
                        x: x - radius,
                        y: y - radius,
                        width: radius * 2,
                        height: radius * 2
                    ))
                    
                    context.fill(circle, with: .color(.systemBlue.opacity(0.3 + Double(i) * 0.15)))
                }
                
                // วาด progress bar
                let barRect = CGRect(x: 20, y: size.height - 40, width: size.width - 40, height: 20)
                
                // Background
                context.fill(
                    Path(roundedRect: barRect, cornerRadius: 10),
                    with: .color(.systemGray4)
                )
                
                // Progress
                let progressWidth = barRect.width * progress
                let progressRect = CGRect(
                    x: barRect.minX,
                    y: barRect.minY,
                    width: progressWidth,
                    height: barRect.height
                )
                
                context.fill(
                    Path(roundedRect: progressRect, cornerRadius: 10),
                    with: .linearGradient(
                        Gradient(colors: [.systemBlue, .systemPurple]),
                        startPoint: .init(x: barRect.minX, y: 0),
                        endPoint: .init(x: barRect.maxX, y: 0)
                    )
                )
            }
            .frame(height: 200)
            
            Slider(value: $progress, in: 0...1)
                .padding()
        }
    }
}

// SwiftUI Canvas ด้วย Symbol Rendering
struct SymbolCanvasExample: View {
    var body: some View {
        Canvas { context, size in
            // วาด text ด้วย Canvas
            let text = Text("Hello, Canvas!")
                .font(.title)
                .bold()
            
            context.draw(text, at: CGPoint(x: size.width / 2, y: 50))
            
            // วาด image ด้วย Canvas
            if let image = context.resolveSymbol(id: "star") {
                context.draw(image, at: CGPoint(x: size.width / 2, y: 120))
            }
            
            // วาด gradient circle
            var circlePath = Path()
            circlePath.addEllipse(in: CGRect(x: 50, y: 150, width: 100, height: 100))
            
            context.fill(
                circlePath,
                with: .radialGradient(
                    Gradient(colors: [.white, .systemBlue]),
                    center: CGPoint(x: 100, y: 200),
                    startRadius: 0,
                    endRadius: 50
                )
            )
        } symbols: {
            Image(systemName: "star.fill")
                .tag("star")
                .foregroundColor(.yellow)
        }
    }
}
```

### เปรียบเทียบ Core Graphics vs SwiftUI Canvas

```swift
// Core Graphics Approach
class CGView: UIView {
    override func draw(_ rect: CGRect) {
        guard let ctx = UIGraphicsGetCurrentContext() else { return }
        ctx.setFillColor(UIColor.systemBlue.cgColor)
        ctx.fillEllipse(in: CGRect(x: 50, y: 50, width: 100, height: 100))
    }
}

// SwiftUI Canvas Approach
struct CanvasView: View {
    var body: some View {
        Canvas { context, _ in
            context.fill(
                Path(ellipseIn: CGRect(x: 50, y: 50, width: 100, height: 100)),
                with: .color(.systemBlue)
            )
        }
    }
}
```

| Feature | Core Graphics | SwiftUI Canvas |
|---------|--------------|----------------|
| Syntax | C-like API | Declarative Swift |
| Performance | เร็วมาก | เร็ว (เกือบเท่า) |
| Dark Mode | จัดการเอง | อัตโนมัติ |
| Accessibility | จัดการเอง | รองรับบางส่วน |
| Platform | iOS/macOS/tvOS | iOS 15+/macOS 12+ |
| Learning Curve | สูง | ต่ำกว่า |

---

## Metal สำหรับ Custom Rendering

**Metal** เป็น low-level GPU programming framework ของ Apple:

```swift
import Metal
import MetalKit
import simd

// Metal View Controller
class MetalViewController: UIViewController {
    
    var device: MTLDevice!
    var commandQueue: MTLCommandQueue!
    var metalView: MTKView!
    var pipelineState: MTLRenderPipelineState!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupMetal()
    }
    
    func setupMetal() {
        // สร้าง Metal device
        guard let device = MTLCreateSystemDefaultDevice() else {
            fatalError("Metal is not supported on this device")
        }
        self.device = device
        
        // สร้าง command queue
        commandQueue = device.makeCommandQueue()
        
        // ตั้งค่า Metal view
        metalView = MTKView(frame: view.bounds, device: device)
        metalView.clearColor = MTLClearColorMake(0.1, 0.1, 0.2, 1.0)
        metalView.delegate = self
        view.addSubview(metalView)
        
        // สร้าง pipeline
        setupPipeline()
    }
    
    func setupPipeline() {
        let library = device.makeDefaultLibrary()!
        
        let vertexFunction = library.makeFunction(name: "vertexShader")
        let fragmentFunction = library.makeFunction(name: "fragmentShader")
        
        let pipelineDescriptor = MTLRenderPipelineDescriptor()
        pipelineDescriptor.vertexFunction = vertexFunction
        pipelineDescriptor.fragmentFunction = fragmentFunction
        pipelineDescriptor.colorAttachments[0].pixelFormat = metalView.colorPixelFormat
        
        pipelineState = try! device.makeRenderPipelineState(descriptor: pipelineDescriptor)
    }
}

extension MetalViewController: MTKViewDelegate {
    
    func mtkView(_ view: MTKView, drawableSizeWillChange size: CGSize) {}
    
    func draw(in view: MTKView) {
        guard let drawable = view.currentDrawable,
              let descriptor = view.currentRenderPassDescriptor else { return }
        
        let commandBuffer = commandQueue.makeCommandBuffer()!
        let encoder = commandBuffer.makeRenderCommandEncoder(descriptor: descriptor)!
        
        encoder.setRenderPipelineState(pipelineState)
        
        // กำหนด vertices (triangle)
        let vertices: [Float] = [
             0.0,  0.5, 0, 1,  // top
            -0.5, -0.5, 0, 1,  // bottom-left
             0.5, -0.5, 0, 1   // bottom-right
        ]
        
        encoder.setVertexBytes(
            vertices,
            length: vertices.count * MemoryLayout<Float>.size,
            index: 0
        )
        
        encoder.drawPrimitives(type: .triangle, vertexStart: 0, vertexCount: 3)
        encoder.endEncoding()
        
        commandBuffer.present(drawable)
        commandBuffer.commit()
    }
}
```

### Metal Shaders (Shaders.metal)

```metal
// Shaders.metal
#include <metal_stdlib>
using namespace metal;

struct VertexOut {
    float4 position [[position]];
    float4 color;
};

vertex VertexOut vertexShader(
    uint vertexID [[vertex_id]],
    constant float4 *vertices [[buffer(0)]]
) {
    VertexOut out;
    out.position = vertices[vertexID];
    
    // กำหนดสีตาม vertex
    float3 colors[3] = {
        float3(1, 0, 0),  // red
        float3(0, 1, 0),  // green
        float3(0, 0, 1)   // blue
    };
    out.color = float4(colors[vertexID], 1.0);
    
    return out;
}

fragment float4 fragmentShader(VertexOut in [[stage_in]]) {
    return in.color;
}
```

---

## SpriteKit Intro

**SpriteKit** เป็น 2D game framework ของ Apple ที่สร้างบน Metal:

```swift
import SpriteKit
import UIKit

// สร้าง Game Scene
class GameScene: SKScene {
    
    private var playerNode: SKSpriteNode!
    private var scoreLabel: SKLabelNode!
    private var score = 0
    
    override func didMove(to view: SKView) {
        setupScene()
        startSpawning()
    }
    
    func setupScene() {
        // Background gradient
        let background = SKSpriteNode(color: .black, size: size)
        background.position = CGPoint(x: size.width / 2, y: size.height / 2)
        addChild(background)
        
        // Player
        playerNode = SKSpriteNode(color: .systemBlue, size: CGSize(width: 60, height: 60))
        playerNode.position = CGPoint(x: size.width / 2, y: 100)
        playerNode.name = "player"
        
        // Physics body
        playerNode.physicsBody = SKPhysicsBody(rectangleOf: playerNode.size)
        playerNode.physicsBody?.isDynamic = false
        playerNode.physicsBody?.categoryBitMask = 0x1
        playerNode.physicsBody?.contactTestBitMask = 0x2
        
        addChild(playerNode)
        
        // Score label
        scoreLabel = SKLabelNode(fontNamed: "Helvetica-Bold")
        scoreLabel.text = "Score: 0"
        scoreLabel.fontSize = 20
        scoreLabel.fontColor = .white
        scoreLabel.position = CGPoint(x: size.width / 2, y: size.height - 60)
        addChild(scoreLabel)
        
        // Physics world
        physicsWorld.gravity = CGVector(dx: 0, dy: -2)
        physicsWorld.contactDelegate = self
    }
    
    func startSpawning() {
        let spawn = SKAction.run { [weak self] in
            self?.spawnEnemy()
        }
        let wait = SKAction.wait(forDuration: 1.5)
        let sequence = SKAction.sequence([spawn, wait])
        let repeatForever = SKAction.repeatForever(sequence)
        
        run(repeatForever)
    }
    
    func spawnEnemy() {
        let enemy = SKShapeNode(circleOfRadius: 20)
        enemy.fillColor = .systemRed
        enemy.strokeColor = .red
        enemy.name = "enemy"
        
        let randomX = CGFloat.random(in: 30...size.width - 30)
        enemy.position = CGPoint(x: randomX, y: size.height + 50)
        
        enemy.physicsBody = SKPhysicsBody(circleOfRadius: 20)
        enemy.physicsBody?.categoryBitMask = 0x2
        
        addChild(enemy)
        
        // สร้าง particle effect
        if let particles = SKEmitterNode(fileNamed: "Spark.sks") {
            particles.position = enemy.position
            particles.targetNode = self
            addChild(particles)
        }
    }
    
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        guard let touch = touches.first else { return }
        let location = touch.location(in: self)
        
        // เลื่อน player ไปตาม touch
        let moveAction = SKAction.moveTo(x: location.x, duration: 0.2)
        playerNode.run(moveAction)
    }
    
    override func update(_ currentTime: TimeInterval) {
        // ลบ enemies ที่ออกนอกหน้าจอ
        enumerateChildNodes(withName: "enemy") { node, _ in
            if node.position.y < -50 {
                node.removeFromParent()
            }
        }
    }
}

extension GameScene: SKPhysicsContactDelegate {
    
    func didBegin(_ contact: SKPhysicsContact) {
        score += 10
        scoreLabel.text = "Score: \(score)"
        
        // Effect เมื่อ collision
        let explosion = SKAction.sequence([
            SKAction.scale(to: 1.5, duration: 0.1),
            SKAction.scale(to: 0, duration: 0.1),
            SKAction.removeFromParent()
        ])
        
        if contact.bodyA.node?.name == "enemy" {
            contact.bodyA.node?.run(explosion)
        } else {
            contact.bodyB.node?.run(explosion)
        }
    }
}

// การตั้งค่า SpriteKit View
class GameViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        guard let skView = view as? SKView else { return }
        
        let scene = GameScene(size: view.bounds.size)
        scene.scaleMode = .aspectFill
        
        skView.showsFPS = true
        skView.showsNodeCount = true
        skView.ignoresSiblingOrder = true
        
        skView.presentScene(scene)
    }
}
```

---

## Practical Exercises

### Exercise 1: วาด Custom Loading Indicator

```swift
import UIKit

class CustomLoadingView: UIView {
    
    private var dots: [CALayer] = []
    private let dotCount = 5
    private let dotSize: CGFloat = 12
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupDots()
        startAnimation()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        setupDots()
        startAnimation()
    }
    
    private func setupDots() {
        for i in 0..<dotCount {
            let dot = CALayer()
            dot.backgroundColor = UIColor.systemBlue.cgColor
            dot.cornerRadius = dotSize / 2
            dot.opacity = 0.3
            layer.addSublayer(dot)
            dots.append(dot)
        }
    }
    
    override func layoutSubviews() {
        super.layoutSubviews()
        
        let spacing = bounds.width / CGFloat(dotCount + 1)
        
        for (index, dot) in dots.enumerated() {
            dot.frame = CGRect(
                x: spacing * CGFloat(index + 1) - dotSize / 2,
                y: bounds.midY - dotSize / 2,
                width: dotSize,
                height: dotSize
            )
        }
    }
    
    private func startAnimation() {
        for (index, dot) in dots.enumerated() {
            let animation = CAKeyframeAnimation(keyPath: "opacity")
            animation.values = [0.3, 1.0, 0.3]
            animation.keyTimes = [0, 0.5, 1.0]
            animation.duration = 1.2
            animation.beginTime = CACurrentMediaTime() + Double(index) * 0.2
            animation.repeatCount = .infinity
            
            dot.add(animation, forKey: "pulse")
        }
    }
    
    func stopAnimation() {
        dots.forEach { $0.removeAllAnimations() }
    }
}
```

### Exercise 2: สร้าง Bar Chart View

```swift
import UIKit

struct BarChartData {
    let label: String
    let value: CGFloat
    let color: UIColor
}

class BarChartView: UIView {
    
    var data: [BarChartData] = [] {
        didSet { setNeedsDisplay() }
    }
    
    var animated: Bool = true
    
    private let padding: CGFloat = 20
    private let labelHeight: CGFloat = 30
    private let animationDuration: TimeInterval = 0.8
    private var animationProgress: CGFloat = 0
    
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext(), !data.isEmpty else { return }
        
        let chartArea = CGRect(
            x: padding,
            y: padding,
            width: rect.width - padding * 2,
            height: rect.height - padding * 2 - labelHeight
        )
        
        // วาด grid lines
        drawGrid(context: context, in: chartArea)
        
        // วาด bars
        let barWidth = chartArea.width / CGFloat(data.count) - 10
        let maxValue = data.map { $0.value }.max() ?? 1
        
        for (index, item) in data.enumerated() {
            let x = chartArea.minX + CGFloat(index) * (barWidth + 10) + 5
            let barHeight = chartArea.height * (item.value / maxValue) * animationProgress
            let y = chartArea.maxY - barHeight
            
            let barRect = CGRect(x: x, y: y, width: barWidth, height: barHeight)
            
            // Draw bar with rounded top
            let path = UIBezierPath(
                roundedRect: barRect,
                byRoundingCorners: [.topLeft, .topRight],
                cornerRadii: CGSize(width: 6, height: 6)
            )
            
            // Gradient fill
            context.saveGState()
            path.addClip()
            
            let colorSpace = CGColorSpaceCreateDeviceRGB()
            let topColor = item.color.cgColor
            let bottomColor = item.color.withAlphaComponent(0.6).cgColor
            
            if let gradient = CGGradient(
                colorsSpace: colorSpace,
                colors: [topColor, bottomColor] as CFArray,
                locations: [0.0, 1.0]
            ) {
                context.drawLinearGradient(
                    gradient,
                    start: CGPoint(x: 0, y: y),
                    end: CGPoint(x: 0, y: y + barHeight),
                    options: []
                )
            }
            
            context.restoreGState()
            
            // Value label บน bar
            if animationProgress > 0.8 {
                let valueText = String(format: "%.0f", item.value)
                let attributes: [NSAttributedString.Key: Any] = [
                    .font: UIFont.boldSystemFont(ofSize: 11),
                    .foregroundColor: item.color
                ]
                let textSize = valueText.size(withAttributes: attributes)
                valueText.draw(
                    at: CGPoint(x: x + barWidth / 2 - textSize.width / 2, y: y - 20),
                    withAttributes: attributes
                )
            }
            
            // Category label
            let labelAttributes: [NSAttributedString.Key: Any] = [
                .font: UIFont.systemFont(ofSize: 11),
                .foregroundColor: UIColor.secondaryLabel
            ]
            let labelSize = item.label.size(withAttributes: labelAttributes)
            item.label.draw(
                at: CGPoint(
                    x: x + barWidth / 2 - labelSize.width / 2,
                    y: chartArea.maxY + 8
                ),
                withAttributes: labelAttributes
            )
        }
    }
    
    private func drawGrid(context: CGContext, in rect: CGRect) {
        let gridLines = 5
        
        for i in 0...gridLines {
            let y = rect.minY + rect.height * CGFloat(i) / CGFloat(gridLines)
            
            context.setStrokeColor(UIColor.separator.cgColor)
            context.setLineWidth(0.5)
            context.setLineDash(phase: 0, lengths: i == gridLines ? [] : [4, 4])
            context.move(to: CGPoint(x: rect.minX, y: y))
            context.addLine(to: CGPoint(x: rect.maxX, y: y))
            context.strokePath()
        }
    }
    
    func animate() {
        animationProgress = 0
        
        let displayLink = CADisplayLink(target: self, selector: #selector(updateAnimation(_:)))
        displayLink.add(to: .main, forMode: .default)
        
        let startTime = CACurrentMediaTime()
        
        Timer.scheduledTimer(withTimeInterval: 1.0 / 60.0, repeats: true) { [weak self, weak displayLink] timer in
            guard let self = self else {
                timer.invalidate()
                return
            }
            
            let elapsed = CACurrentMediaTime() - startTime
            let fraction = min(elapsed / self.animationDuration, 1.0)
            
            // Ease out
            self.animationProgress = 1 - pow(1 - fraction, 3)
            self.setNeedsDisplay()
            
            if fraction >= 1.0 {
                timer.invalidate()
                displayLink?.invalidate()
            }
        }
    }
    
    @objc private func updateAnimation(_ displayLink: CADisplayLink) {
        setNeedsDisplay()
    }
}

// ViewController การใช้งาน
class ChartViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let chartView = BarChartView(frame: CGRect(x: 20, y: 100, width: 350, height: 300))
        chartView.backgroundColor = .systemBackground
        chartView.layer.cornerRadius = 16
        chartView.layer.shadowColor = UIColor.black.cgColor
        chartView.layer.shadowOpacity = 0.1
        chartView.layer.shadowRadius = 8
        chartView.layer.shadowOffset = CGSize(width: 0, height: 4)
        
        chartView.data = [
            BarChartData(label: "ม.ค.", value: 120, color: .systemBlue),
            BarChartData(label: "ก.พ.", value: 85, color: .systemGreen),
            BarChartData(label: "มี.ค.", value: 150, color: .systemOrange),
            BarChartData(label: "เม.ย.", value: 95, color: .systemPurple),
            BarChartData(label: "พ.ค.", value: 180, color: .systemRed)
        ]
        
        view.addSubview(chartView)
        
        DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
            chartView.animate()
        }
    }
}
```

---

## Building a Custom Chart/Drawing App

สร้าง Drawing App ที่ใช้ Core Graphics และ UIBezierPath:

```swift
import UIKit

// MARK: - Drawing Tool Enum
enum DrawingTool {
    case pen
    case eraser
    case rectangle
    case circle
    case line
}

// MARK: - Stroke Model
struct Stroke {
    var path: UIBezierPath
    var color: UIColor
    var lineWidth: CGFloat
    var isFilled: Bool
}

// MARK: - Drawing Canvas View
class DrawingCanvasView: UIView {
    
    // MARK: - Properties
    var currentTool: DrawingTool = .pen
    var currentColor: UIColor = .systemBlue
    var currentLineWidth: CGFloat = 3.0
    var currentFill: Bool = false
    
    private var strokes: [Stroke] = []
    private var redoStack: [Stroke] = []
    private var currentPath: UIBezierPath?
    private var startPoint: CGPoint = .zero
    private var lastPoint: CGPoint = .zero
    
    // Off-screen buffer สำหรับ performance
    private var bufferImage: UIImage?
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        backgroundColor = .white
        isMultipleTouchEnabled = false
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        backgroundColor = .white
    }
    
    // MARK: - Drawing
    
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext() else { return }
        
        // วาด buffer image (strokes ก่อนหน้า)
        bufferImage?.draw(in: bounds)
        
        // วาด current stroke
        if let path = currentPath {
            context.setStrokeColor(currentColor.cgColor)
            context.setFillColor(currentColor.cgColor)
            context.setLineWidth(currentLineWidth)
            context.setLineCap(.round)
            context.setLineJoin(.round)
            
            switch currentTool {
            case .pen, .eraser, .line:
                context.addPath(path.cgPath)
                context.strokePath()
                
            case .rectangle, .circle:
                if currentFill {
                    context.addPath(path.cgPath)
                    context.fillPath()
                } else {
                    context.addPath(path.cgPath)
                    context.strokePath()
                }
            }
        }
    }
    
    // MARK: - Touch Handling
    
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        guard let touch = touches.first else { return }
        
        startPoint = touch.location(in: self)
        lastPoint = startPoint
        redoStack.removeAll()
        
        switch currentTool {
        case .pen, .eraser:
            currentPath = UIBezierPath()
            currentPath?.lineCapStyle = .round
            currentPath?.lineJoinStyle = .round
            currentPath?.move(to: startPoint)
            
        case .rectangle, .circle, .line:
            currentPath = UIBezierPath()
        }
    }
    
    override func touchesMoved(_ touches: Set<UITouch>, with event: UIEvent?) {
        guard let touch = touches.first else { return }
        
        let currentPoint = touch.location(in: self)
        
        switch currentTool {
        case .pen:
            // Smooth line ด้วย midpoint algorithm
            let midPoint = CGPoint(
                x: (lastPoint.x + currentPoint.x) / 2,
                y: (lastPoint.y + currentPoint.y) / 2
            )
            currentPath?.addQuadCurve(to: midPoint, controlPoint: lastPoint)
            lastPoint = currentPoint
            
        case .eraser:
            currentPath?.addLine(to: currentPoint)
            lastPoint = currentPoint
            
        case .rectangle:
            let rect = CGRect(
                x: min(startPoint.x, currentPoint.x),
                y: min(startPoint.y, currentPoint.y),
                width: abs(currentPoint.x - startPoint.x),
                height: abs(currentPoint.y - startPoint.y)
            )
            currentPath = UIBezierPath(rect: rect)
            
        case .circle:
            let rect = CGRect(
                x: min(startPoint.x, currentPoint.x),
                y: min(startPoint.y, currentPoint.y),
                width: abs(currentPoint.x - startPoint.x),
                height: abs(currentPoint.y - startPoint.y)
            )
            currentPath = UIBezierPath(ovalIn: rect)
            
        case .line:
            currentPath = UIBezierPath()
            currentPath?.move(to: startPoint)
            currentPath?.addLine(to: currentPoint)
        }
        
        setNeedsDisplay()
    }
    
    override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
        guard let path = currentPath else { return }
        
        // บันทึก stroke ใน buffer
        let stroke = Stroke(
            path: path,
            color: currentTool == .eraser ? .white : currentColor,
            lineWidth: currentLineWidth,
            isFilled: currentFill
        )
        
        strokes.append(stroke)
        
        // Render ลง buffer
        renderToBuffer()
        
        currentPath = nil
        setNeedsDisplay()
    }
    
    // MARK: - Buffer Management
    
    private func renderToBuffer() {
        let renderer = UIGraphicsImageRenderer(bounds: bounds)
        
        bufferImage = renderer.image { context in
            // วาด buffer เดิม
            bufferImage?.draw(in: bounds)
            
            // วาด stroke ใหม่
            if let lastStroke = strokes.last {
                let ctx = context.cgContext
                ctx.setStrokeColor(lastStroke.color.cgColor)
                ctx.setFillColor(lastStroke.color.cgColor)
                ctx.setLineWidth(lastStroke.lineWidth)
                ctx.setLineCap(.round)
                ctx.setLineJoin(.round)
                
                if lastStroke.isFilled {
                    ctx.addPath(lastStroke.path.cgPath)
                    ctx.fillPath()
                } else {
                    ctx.addPath(lastStroke.path.cgPath)
                    ctx.strokePath()
                }
            }
        }
    }
    
    // MARK: - Actions
    
    func undo() {
        guard let lastStroke = strokes.popLast() else { return }
        redoStack.append(lastStroke)
        rebuildBuffer()
        setNeedsDisplay()
    }
    
    func redo() {
        guard let stroke = redoStack.popLast() else { return }
        strokes.append(stroke)
        renderToBuffer()
        setNeedsDisplay()
    }
    
    func clear() {
        strokes.removeAll()
        redoStack.removeAll()
        bufferImage = nil
        setNeedsDisplay()
    }
    
    private func rebuildBuffer() {
        bufferImage = nil
        
        guard !strokes.isEmpty else { return }
        
        let renderer = UIGraphicsImageRenderer(bounds: bounds)
        
        bufferImage = renderer.image { context in
            let ctx = context.cgContext
            
            for stroke in strokes {
                ctx.setStrokeColor(stroke.color.cgColor)
                ctx.setFillColor(stroke.color.cgColor)
                ctx.setLineWidth(stroke.lineWidth)
                ctx.setLineCap(.round)
                ctx.setLineJoin(.round)
                
                if stroke.isFilled {
                    ctx.addPath(stroke.path.cgPath)
                    ctx.fillPath()
                } else {
                    ctx.addPath(stroke.path.cgPath)
                    ctx.strokePath()
                }
            }
        }
    }
    
    func exportAsImage() -> UIImage? {
        let renderer = UIGraphicsImageRenderer(bounds: bounds)
        
        return renderer.image { _ in
            bufferImage?.draw(in: bounds)
        }
    }
}

// MARK: - Drawing View Controller
class DrawingViewController: UIViewController {
    
    private let canvas = DrawingCanvasView()
    private let toolbar = UIToolbar()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
    }
    
    private func setupUI() {
        title = "Drawing App"
        view.backgroundColor = .systemBackground
        
        // Canvas
        canvas.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(canvas)
        
        NSLayoutConstraint.activate([
            canvas.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            canvas.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            canvas.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            canvas.bottomAnchor.constraint(equalTo: view.bottomAnchor, constant: -100)
        ])
        
        // Toolbar
        setupToolbar()
    }
    
    private func setupToolbar() {
        let penButton = UIBarButtonItem(image: UIImage(systemName: "pencil"),
                                        style: .plain, target: self, action: #selector(selectPen))
        let eraserButton = UIBarButtonItem(image: UIImage(systemName: "eraser"),
                                           style: .plain, target: self, action: #selector(selectEraser))
        let undoButton = UIBarButtonItem(image: UIImage(systemName: "arrow.uturn.backward"),
                                          style: .plain, target: self, action: #selector(undo))
        let redoButton = UIBarButtonItem(image: UIImage(systemName: "arrow.uturn.forward"),
                                          style: .plain, target: self, action: #selector(redo))
        let clearButton = UIBarButtonItem(image: UIImage(systemName: "trash"),
                                           style: .plain, target: self, action: #selector(clear))
        let shareButton = UIBarButtonItem(image: UIImage(systemName: "square.and.arrow.up"),
                                           style: .plain, target: self, action: #selector(share))
        
        let flex = UIBarButtonItem(barButtonSystemItem: .flexibleSpace, target: nil, action: nil)
        
        navigationItem.rightBarButtonItems = [shareButton, clearButton, redoButton, undoButton]
        navigationItem.leftBarButtonItems = [penButton, eraserButton]
        
        _ = flex  // suppress warning
    }
    
    @objc func selectPen() { canvas.currentTool = .pen }
    @objc func selectEraser() { canvas.currentTool = .eraser }
    @objc func undo() { canvas.undo() }
    @objc func redo() { canvas.redo() }
    @objc func clear() { canvas.clear() }
    
    @objc func share() {
        guard let image = canvas.exportAsImage() else { return }
        
        let activityVC = UIActivityViewController(
            activityItems: [image],
            applicationActivities: nil
        )
        
        present(activityVC, animated: true)
    }
}
```

---

## สรุป

ในบทนี้เราได้เรียนรู้เกี่ยวกับ Core Graphics และ Drawing ใน iOS อย่างครอบคลุม:

### สิ่งที่ได้เรียนรู้

1. **Core Graphics (Quartz 2D)**
   - CGContext เป็น canvas สำหรับวาด
   - State machine model (save/restore)
   - Path-based drawing

2. **Shape Drawing**
   - CGPath สำหรับ geometric shapes
   - UIBezierPath สำหรับ high-level drawing
   - Custom shapes (star, arrow, speech bubble)

3. **Visual Effects**
   - Gradients (linear, radial, conic)
   - Shadows
   - Transformations (translate, rotate, scale)

4. **Layer System**
   - CALayer เป็น foundation
   - CAShapeLayer สำหรับ vector graphics
   - CAGradientLayer สำหรับ gradients
   - CATextLayer สำหรับ text

5. **Animations**
   - CABasicAnimation
   - CAKeyframeAnimation
   - CASpringAnimation
   - CAAnimationGroup

6. **Modern Approaches**
   - SwiftUI Canvas สำหรับ declarative drawing
   - Metal สำหรับ GPU-accelerated rendering
   - SpriteKit สำหรับ 2D games

### เมื่อควรใช้อะไร

```
Custom UIView draw(_:)  → Simple, infrequent drawing
CALayer                 → Performance-critical, animations
SwiftUI Canvas         → SwiftUI apps, iOS 15+
Metal                  → Complex graphics, games, AR
SpriteKit              → 2D games
```

### Tips สำหรับ Performance

```swift
// ✅ ใช้ setNeedsDisplay() แทน direct drawing
view.setNeedsDisplay()

// ✅ ใช้ UIGraphicsImageRenderer
let renderer = UIGraphicsImageRenderer(size: size)
let image = renderer.image { _ in /* draw */ }

// ✅ Cache buffer image สำหรับ complex drawings
var bufferImage: UIImage?

// ✅ ใช้ isOpaque = true เมื่อไม่ต้องการ transparency
view.isOpaque = true
view.backgroundColor = .white

// ❌ หลีกเลี่ยงการวาดใน override draw() บ่อยๆ
// ❌ หลีกเลี่ยง CALayer ที่ซ้อนกันลึกมาก
// ❌ หลีกเลี่ยง masksToBounds ถ้าไม่จำเป็น
```

---

*Part 64 จบ - ไปต่อ Part 65*
