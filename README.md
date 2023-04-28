# swiftui-multiple-imagepicker

```swift

import Foundation
import SwiftUI
import PhotosUI

struct MultipleImagePicker: UIViewControllerRepresentable {
    
    let sourceType: UIImagePickerController.SourceType
    let onSelectImages: ([UIImage]) -> Void
    let onCancel: () -> Void
    
    func makeUIViewController(context: Context) -> PHPickerViewController {
        var configuration = PHPickerConfiguration()
        configuration.selectionLimit = 36 // Selection limit. Set to 0 for unlimited.
        configuration.filter = .images // he types of media that can be get.
        let picker = PHPickerViewController(configuration: configuration)
        picker.delegate = context.coordinator
        return picker
    }
    
    func updateUIViewController(_ uiViewController: PHPickerViewController, context: Context) {
        
    }
    
    func makeCoordinator() -> MultipleImagePickerCoordinator {
        return MultipleImagePickerCoordinator(onSelectImages: self.onSelectImages, onCancel: self.onCancel)
    }
    
    class MultipleImagePickerCoordinator: NSObject, PHPickerViewControllerDelegate {
        
        private let onSelectImages: ([UIImage]) -> Void
        private let onCancel: () -> Void
        
        init(onSelectImages: @escaping([UIImage]) -> Void, onCancel: @escaping() -> Void) {
            self.onSelectImages = onSelectImages
            self.onCancel = onCancel
        }
        
        func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {
            picker.dismiss(animated: true)
            
            var images = [UIImage]()
            
            for result in results {
                let itemProvider = result.itemProvider
                guard itemProvider.canLoadObject(ofClass: UIImage.self) else { continue }
                itemProvider.loadObject(ofClass: UIImage.self) { image, error in
                    if let error {
                        Log.error(error)
                    }
                    if let image = image as? UIImage {
                        images.append(image)
                    }
                }
            }
            
            self.onSelectImages(images)
            
        }
        
    }
    
}


```
