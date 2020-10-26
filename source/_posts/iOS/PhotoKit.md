---
title:  PhotoKit 
date:  2019-03-13 11:13
categories:
- iOS
tags: 
- PhotoKit 
---
![Photos.png](https://upload-images.jianshu.io/upload_images/3340896-b7a0956c45c82900.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 介绍
苹果的PhotoKit,是提供给开发者的对本地相册库的和iCloud 相册的资源进行操作的API,所有资源以PHAsset的形式来提供给PhotoKit使用,同时本地的图片库和iCloud图片的变动通知,会发送给PhotoKit;同时,PhotoKit也可以通过 **变更请求** (编辑请求,改变请求,删除请求...)来对资源进行变更.

### 资源的操作
```
PHPhotoLibrary.shared().register(self); // viewDidLoad 执行
PHPhotoLibrary.shared().unregisterChangeObserver(self);//deinit 方法中执行

extension MasterViewController: PHPhotoLibraryChangeObserver {//资源变动后的通知
    /// - Tag: RespondToChanges
    func photoLibraryDidChange(_ changeInstance: PHChange) {
        
        // Change notifications may originate from a background queue.
        // Re-dispatch to the main queue before acting on the change,
        // so you can update the UI.
        DispatchQueue.main.sync {
        }
    }
}

```

### 资源的查找
排序和筛选条件我们可以根据 [PHFetchOptions](https://developer.apple.com/documentation/photokit/phfetchoptions)里面的 `predicate` 和 `sortDescriptors` 属性来设置, 里面也包含支持 **predicate** 和 **sort** 的 keys, 可以参考 [NSPredicate](https://nshipster.cn/nspredicate/) 和  [NSSortDescriptor](https://nshipster.cn/nssortdescriptor/) 来了解更多筛选和排序条件的设置.
```
PHAssetCollection.fetchAssetCollections(with: .smartAlbum, subtype: .albumRegular, options: nil)//智能相册
PHCollectionList.fetchTopLevelUserCollections(with: nil)//用户相册
PHAsset.fetchAssets(with: phtotfectchOptions)//图片资源

```

### 资源的变更请求
```
//Asynchronously runs a block that requests changes to be performed in the photo library.  异步请求
func  performChanges(_:completionHandler:)
//Synchronously runs a block that requests changes to be performed in the photo library.  同步请求
func performChangesAndWait(_ changeBlock: @escaping () -> Void) throws
```

### Album的操作(增加,修改,删除)
```
//增加相册
func addAlbum() {
        PHPhotoLibrary.shared().performChanges({
            let assetCollectionRequest = PHCollectionListChangeRequest.creationRequestForCollectionList(withTitle: "title");
            self.identifier =  assetCollectionRequest.placeholderForCreatedCollectionList.localIdentifier;
        }) { (ret, error) in
            if ret {
                if let id = self.identifier {
                    let collections = PHCollectionList.fetchCollectionLists(withLocalIdentifiers: [id], options: nil)
                    print("\(collections)");
                }
            }
        }
    }
    
//删除相册
 func deleteAlbum() {
        let fecthOptions = PHFetchOptions();
        let predicate = NSPredicate.init(format: "localizedTitle == 'title'");
        fecthOptions.predicate = predicate;
        let titleList = PHCollectionList.fetchCollectionLists(with: .folder, subtype: .regularFolder, options: fecthOptions);
        
        PHPhotoLibrary.shared().performChanges({
            PHCollectionListChangeRequest.deleteCollectionLists(titleList);
        }) { (ret, error) in
            if ret {
                print("删除成功");
            }
        }
    }
//修改相册
  func modifyAlbum() {
        let fecthOptions = PHFetchOptions();
        let predicate = NSPredicate.init(format: "localizedTitle == 'title'");
        fecthOptions.predicate = predicate;
        let titleList = PHCollectionList.fetchCollectionLists(with: .folder, subtype: .regularFolder, options: fecthOptions);
        PHPhotoLibrary.shared().performChanges({
            
            if let collection = titleList.firstObject {
                let modifyRequest = PHCollectionListChangeRequest(for: collection);
                modifyRequest?.title = "modify album";
            }
        }) { (ret, error) in
            if ret {
                print("修改成功");
            }
        }
   }
```
###  Asset的操作
```
   //增加
   func creationRequestForAsset(from image: UIImage) -> Self 
   func creationRequestForAssetFromImage(atFileURL fileURL: URL) -> Self?
   func creationRequestForAssetFromVideo(atFileURL fileURL: URL) -> Self?
   //删除
   func deleteAssets(_ assets: NSFastEnumeration)
   //修改
  func requestContentEditingInput(with options: PHContentEditingInputRequestOptions?, completionHandler: @escaping (PHContentEditingInput?, [AnyHashable : Any]) -> Void) -> PHContentEditingInputRequestID
  func cancelContentEditingInputRequest(_ requestID: PHContentEditingInputRequestID)

   // MARK: UI Actions
    /// - Tag: EditAlert
    @IBAction func editAsset(_ sender: UIBarButtonItem) {
        // Use a UIAlertController to display editing options to the user.
        let alertController = UIAlertController(title: nil, message: nil, preferredStyle: .actionSheet)
        #if os(iOS)
        alertController.modalPresentationStyle = .popover
        if let popoverController = alertController.popoverPresentationController {
            popoverController.barButtonItem = sender
            popoverController.permittedArrowDirections = .up
        }
        #endif
        
        // Add a Cancel action to dismiss the alert without doing anything.
        alertController.addAction(UIAlertAction(title: NSLocalizedString("Cancel", comment: ""),
                                                style: .cancel, handler: nil))
        // Allow editing only if the PHAsset supports edit operations.
        if asset.canPerform(.content) {
            // Add actions for some canned filters.
            alertController.addAction(UIAlertAction(title: NSLocalizedString("Sepia Tone", comment: ""),
                                                    style: .default, handler: getFilter("CISepiaTone")))
            alertController.addAction(UIAlertAction(title: NSLocalizedString("Chrome", comment: ""),
                                                    style: .default, handler: getFilter("CIPhotoEffectChrome")))
            
            // Add actions to revert any edits that have been made to the PHAsset.
            alertController.addAction(UIAlertAction(title: NSLocalizedString("Revert", comment: ""),
                                                    style: .default, handler: revertAsset))
        }
        // Present the UIAlertController.
        present(alertController, animated: true)
    }
    #if os(tvOS)
    @IBAction func playLivePhoto(_ sender: Any) {
        livePhotoView.startPlayback(with: .full)
    }
    #endif
    /// - Tag: PlayVideo
    @IBAction func play(_ sender: AnyObject) {
        if playerLayer != nil {
            // The app already created an AVPlayerLayer, so tell it to play.
            playerLayer.player!.play()
        } else {
            let options = PHVideoRequestOptions()
            options.isNetworkAccessAllowed = true
            options.deliveryMode = .automatic
            options.progressHandler = { progress, _, _, _ in
                // The handler may originate on a background queue, so
                // re-dispatch to the main queue for UI work.
                DispatchQueue.main.sync {
                    self.progressView.progress = Float(progress)
                }
            }
            // Request an AVPlayerItem for the displayed PHAsset.
            // Then configure a layer for playing it.
            PHImageManager.default().requestPlayerItem(forVideo: asset, options: options, resultHandler: { playerItem, info in
                DispatchQueue.main.sync {
                    guard self.playerLayer == nil else { return }
                    
                    // Create an AVPlayer and AVPlayerLayer with the AVPlayerItem.
                    let player = AVPlayer(playerItem: playerItem)
                    let playerLayer = AVPlayerLayer(player: player)
                    
                    // Configure the AVPlayerLayer and add it to the view.
                    playerLayer.videoGravity = AVLayerVideoGravity.resizeAspect
                    playerLayer.frame = self.view.layer.bounds
                    self.view.layer.addSublayer(playerLayer)
                    
                    player.play()
                    
                    // Cache the player layer by reference, so you can remove it later.
                    self.playerLayer = playerLayer
                }
            })
        }
    }
    /// - Tag: RemoveAsset
    @IBAction func removeAsset(_ sender: AnyObject) {
        let completion = { (success: Bool, error: Error?) -> Void in
            if success {
                PHPhotoLibrary.shared().unregisterChangeObserver(self)
                DispatchQueue.main.sync {
                    _ = self.navigationController!.popViewController(animated: true)
                }
            } else {
                print("Can't remove the asset: \(String(describing: error))")
            }
        }
        if assetCollection != nil {
            // Remove the asset from the selected album.
            PHPhotoLibrary.shared().performChanges({
                let request = PHAssetCollectionChangeRequest(for: self.assetCollection)!
                request.removeAssets([self.asset] as NSArray)
            }, completionHandler: completion)
        } else {
            // Delete the asset from the photo library.
            PHPhotoLibrary.shared().performChanges({
                PHAssetChangeRequest.deleteAssets([self.asset] as NSArray)
            }, completionHandler: completion)
        }
    }
    /// - Tag: MarkFavorite
    @IBAction func toggleFavorite(_ sender: UIBarButtonItem) {
        PHPhotoLibrary.shared().performChanges({
            let request = PHAssetChangeRequest(for: self.asset)
            request.isFavorite = !self.asset.isFavorite
        }, completionHandler: { success, error in
            if success {
                DispatchQueue.main.sync {
                    sender.title = self.asset.isFavorite ? "♥︎" : "♡"
                }
            } else {
                print("Can't mark the asset as a Favorite: \(String(describing: error))")
            }
        })
    }
    
    // MARK: Image display
    
    var targetSize: CGSize {
        let scale = UIScreen.main.scale
        return CGSize(width: imageView.bounds.width * scale, height: imageView.bounds.height * scale)
    }
    
    func updateImage() {
        if asset.mediaSubtypes.contains(.photoLive) {
            updateLivePhoto()
        } else {
            updateStaticImage()
        }
    }
    
    func updateLivePhoto() {
        // Prepare the options to pass when fetching the live photo.
        let options = PHLivePhotoRequestOptions()
        options.deliveryMode = .highQualityFormat
        options.isNetworkAccessAllowed = true
        options.progressHandler = { progress, _, _, _ in
            // The handler may originate on a background queue, so
            // re-dispatch to the main queue for UI work.
            DispatchQueue.main.sync {
                self.progressView.progress = Float(progress)
            }
        }
        
        // Request the live photo for the asset from the default PHImageManager.
        PHImageManager.default().requestLivePhoto(for: asset, targetSize: targetSize, contentMode: .aspectFit, options: options,
                                                  resultHandler: { livePhoto, info in
                                                    // PhotoKit finishes the request, so hide the progress view.
                                                    self.progressView.isHidden = true
                                                    
                                                    // Show the Live Photo view.
                                                    guard let livePhoto = livePhoto else { return }
                                                    
                                                    // Show the Live Photo.
                                                    self.imageView.isHidden = true
                                                    self.livePhotoView.isHidden = false
                                                    self.livePhotoView.livePhoto = livePhoto
                                                    
                                                    if !self.isPlayingHint {
                                                        // Play back a short section of the Live Photo, similar to the Photos share sheet.
                                                        self.isPlayingHint = true
                                                        self.livePhotoView.startPlayback(with: .hint)
                                                    }
        })
    }
    
    func updateStaticImage() {
        // Prepare the options to pass when fetching the (photo, or video preview) image.
        let options = PHImageRequestOptions()
        options.deliveryMode = .highQualityFormat
        options.isNetworkAccessAllowed = true
        options.progressHandler = { progress, _, _, _ in
            // The handler may originate on a background queue, so
            // re-dispatch to the main queue for UI work.
            DispatchQueue.main.sync {
                self.progressView.progress = Float(progress)
            }
        }
        
        PHImageManager.default().requestImage(for: asset, targetSize: targetSize, contentMode: .aspectFit, options: options,
                                              resultHandler: { image, _ in
                                                // PhotoKit finished the request, so hide the progress view.
                                                self.progressView.isHidden = true
                                                
                                                // If the request succeeded, show the image view.
                                                guard let image = image else { return }
                                                
                                                // Show the image.
                                                self.livePhotoView.isHidden = true
                                                self.imageView.isHidden = false
                                                self.imageView.image = image
        })
    }
    
    // MARK: Asset editing
    
    func revertAsset(sender: UIAlertAction) {
        PHPhotoLibrary.shared().performChanges({
            let request = PHAssetChangeRequest(for: self.asset)
            request.revertAssetContentToOriginal()
        }, completionHandler: { success, error in
            if !success { print("Can't revert the asset: \(String(describing: error))") }
        })
    }
    
    // Returns a filter-applier function for the named filter.
    // Use the function as a handler for a UIAlertAction object.
    /// - Tag: ApplyFilter
    func getFilter(_ filterName: String) -> (UIAlertAction) -> Void {
        func applyFilter(_: UIAlertAction) {
            // Set up a handler to handle prior edits.
            let options = PHContentEditingInputRequestOptions()
            options.canHandleAdjustmentData = {
                $0.formatIdentifier == self.formatIdentifier && $0.formatVersion == self.formatVersion
            }
            
            // Prepare for editing.
            asset.requestContentEditingInput(with: options, completionHandler: { input, info in
                guard let input = input
                    else { fatalError("Can't get the content-editing input: \(info)") }
                
                // This handler executes on the main thread; dispatch to a background queue for processing.
                DispatchQueue.global(qos: .userInitiated).async {
                    
                    // Create adjustment data describing the edit.
                    let adjustmentData = PHAdjustmentData(formatIdentifier: self.formatIdentifier,
                                                          formatVersion: self.formatVersion,
                                                          data: filterName.data(using: .utf8)!)
                    
                    // Create content editing output, write the adjustment data.
                    let output = PHContentEditingOutput(contentEditingInput: input)
                    output.adjustmentData = adjustmentData
                    
                    // Select a filtering function for the asset's media type.
                    let applyFunc: (String, PHContentEditingInput, PHContentEditingOutput, @escaping () -> Void) -> Void
                    if self.asset.mediaSubtypes.contains(.photoLive) {
                        applyFunc = self.applyLivePhotoFilter
                    } else if self.asset.mediaType == .image {
                        applyFunc = self.applyPhotoFilter
                    } else {
                        applyFunc = self.applyVideoFilter
                    }
                    
                    // Apply the filter.
                    applyFunc(filterName, input, output, {
                        // When the app finishes rendering the filtered result, commit the edit to the photo library.
                        PHPhotoLibrary.shared().performChanges({
                            let request = PHAssetChangeRequest(for: self.asset)
                            request.contentEditingOutput = output
                        }, completionHandler: { success, error in
                            if !success { print("Can't edit the asset: \(String(describing: error))") }
                        })
                    })
                }
            })
        }
        return applyFilter
    }
    
    func applyPhotoFilter(_ filterName: String, input: PHContentEditingInput, output: PHContentEditingOutput, completion: () -> Void) {
        
        // Load the full-size image.
        guard let inputImage = CIImage(contentsOf: input.fullSizeImageURL!)
            else { fatalError("Can't load the input image to edit.") }
        
        // Apply the filter.
        let outputImage = inputImage
            .oriented(forExifOrientation: input.fullSizeImageOrientation)
            .applyingFilter(filterName, parameters: [:])
        
        // Write the edited image as a JPEG.
        do {
            try self.ciContext.writeJPEGRepresentation(of: outputImage,
                                                       to: output.renderedContentURL, colorSpace: inputImage.colorSpace!, options: [:])
        } catch let error {
            fatalError("Can't apply the filter to the image: \(error).")
        }
        completion()
    }
    
    func applyLivePhotoFilter(_ filterName: String, input: PHContentEditingInput, output: PHContentEditingOutput, completion: @escaping () -> Void) {
        
        // This app filters assets only for output. In an app that previews
        // filters while editing, create a livePhotoContext early and reuse it
        // to render both for previewing and for final output.
        guard let livePhotoContext = PHLivePhotoEditingContext(livePhotoEditingInput: input)
            else { fatalError("Can't fetch the Live Photo to edit.") }
        
        livePhotoContext.frameProcessor = { frame, _ in
            return frame.image.applyingFilter(filterName, parameters: [:])
        }
        livePhotoContext.saveLivePhoto(to: output) { success, error in
            if success {
                completion()
            } else {
                print("Can't output the Live Photo.")
            }
        }
    }
    
    func applyVideoFilter(_ filterName: String, input: PHContentEditingInput, output: PHContentEditingOutput, completion: @escaping () -> Void) {
        // Load the AVAsset to process from input.
        guard let avAsset = input.audiovisualAsset
            else { fatalError("Can't fetch the AVAsset to edit.") }
        
        // Set up a video composition to apply the filter.
        let composition = AVVideoComposition(
            asset: avAsset,
            applyingCIFiltersWithHandler: { request in
                let filtered = request.sourceImage.applyingFilter(filterName, parameters: [:])
                request.finish(with: filtered, context: nil)
        })
        
        // Export the video composition to the output URL.
        guard let export = AVAssetExportSession(asset: avAsset, presetName: AVAssetExportPresetHighestQuality)
            else { fatalError("Can't configure the AVAssetExportSession.") }
        export.outputFileType = AVFileType.mov
        export.outputURL = output.renderedContentURL
        export.videoComposition = composition
        export.exportAsynchronously(completionHandler: completion)
    }

// MARK: PHPhotoLibraryChangeObserver
extension AssetViewController: PHPhotoLibraryChangeObserver {
    func photoLibraryDidChange(_ changeInstance: PHChange) {
        // The call might come on any background queue. Re-dispatch to the main queue to handle it.
        DispatchQueue.main.sync {
            // Check if there are changes to the displayed asset.
            guard let details = changeInstance.changeDetails(for: asset) else { return }
            
            // Get the updated asset.
            asset = details.objectAfterChanges
            
            // If the asset's content changes, update the image and stop any video playback.
            if details.assetContentChanged {
                updateImage()
                
                playerLayer?.removeFromSuperlayer()
                playerLayer = nil
            }
        }
    }
}

// MARK: PHLivePhotoViewDelegate
extension AssetViewController: PHLivePhotoViewDelegate {
    func livePhotoView(_ livePhotoView: PHLivePhotoView, willBeginPlaybackWith playbackStyle: PHLivePhotoViewPlaybackStyle) {
        isPlayingHint = (playbackStyle == .hint)
    }
    
    func livePhotoView(_ livePhotoView: PHLivePhotoView, didEndPlaybackWith playbackStyle: PHLivePhotoViewPlaybackStyle) {
        isPlayingHint = (playbackStyle == .hint)
    }
}

```

## 参考资料
[Browsing and Modifying Photo Albums](https://developer.apple.com/documentation/photokit/browsing_and_modifying_photo_albums)
[NSPredicate](https://nshipster.cn/nspredicate/)
[NSSortDescriptor](https://nshipster.cn/nssortdescriptor/)
