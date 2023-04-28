# swiftui-multiple-imagepicker

```swift
//
//  MultipleImagePicker.swift
//  ProteinTracker
//
//  Created by paige shin on 2023/04/29.
//

import Foundation
import SwiftUI
import PhotosUI

struct MultipleImagePicker: UIViewControllerRepresentable {
    
    let onSelectImages: ([UIImage]) -> Void
    let onCancel: () -> Void
    var selectionLimit = 36
    
    func makeUIViewController(context: Context) -> PHPickerViewController {
        var configuration = PHPickerConfiguration()
        configuration.selectionLimit = self.selectionLimit // Selection limit. Set to 0 for unlimited.
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
            Task {
                var images = [UIImage]()
                for result in results {
                    do {
                        if let image = try await self.loadImage(provider: result.itemProvider) {
                            images.append(image)
                        }
                    } catch {
                        Log.error(error)
                        continue
                    }
                }
                self.onSelectImages(images)
            }
        }
        
        private func loadImage(provider: NSItemProvider) async throws -> UIImage? {
            return try await withCheckedThrowingContinuation { continuation in
                guard provider.canLoadObject(ofClass: UIImage.self) else {
                    continuation.resume(returning: nil)
                    return
                }
                provider.loadObject(ofClass: UIImage.self) { image, error in
                    if let error {
                        continuation.resume(throwing: error)
                        return
                    }
                    if let image = image as? UIImage {
                        continuation.resume(returning: image)
                        return
                    }
                    continuation.resume(returning: nil)
                }
            }
        }
        
    }
    
}



```
