# Unity でのロケータブル カメラ

## 目次

-   [1 フォトビデオカメラの機能の有効化](Locatable%20camera%20in%20Unity.md#フォトビデオカメラの機能の有効化)

-   [2 写真のキャプチャ](Locatable%20camera%20in%20Unity.md#写真のキャプチャ)

    -   [2.1 PhotoCaptureの共通セットアップ](Locatable%20camera%20in%20Unity.md#photocapture-の共通セットアップ)

    -   [2.2 ファイルへの写真のキャプチャ](Locatable%20camera%20in%20Unity.md#ファイルへの写真のキャプチャ)

    -   [2.3 Texture2Dへの写真のキャプチャ](Locatable%20camera%20in%20Unity.md#texture2d-への写真のキャプチャ)

    -   [2.4 写真のキャプチャとキャプチャしたバイト列そのものの操作](Locatable%20camera%20in%20Unity.md#写真のキャプチャとキャプチャしたバイト列そのものの操作)

-   [3 ビデオのキャプチャ](Locatable%20camera%20in%20Unity.md#ビデオのキャプチャ)

-   [4 トラブルシューティング](Locatable%20camera%20in%20Unity.md#トラブルシューティング)

-   [5 関連項目](Locatable%20camera%20in%20Unity.md#関連項目)

## フォトビデオカメラの機能の有効化

アプリで[*カメラ*](https://developer.microsoft.com/ja-jp/windows/mixed-reality/locatable_camera)を使用するには「Webカメラ」機能を宣言する必要あります。

1.  Unity エディターで、[Edit] (編集)、[Project Settings] (プロジェクト設定)、[Player] (プレイヤー)
    の順に選び、プレイヤーの設定に移動します。

2.  [Windows Store] (Windows ストア) タブをクリックします。

3.  [Publishing Settings] (発行の設定) の [Capabilities] (機能)セクションで、**[WebCam]** (Web カメラ)機能と **[Microphone]** (マイク)機能のチェック ボックスをオンにします。

カメラでは一度に 1 つの操作しか実行できません。カメラの現在モード(フォト、ビデオ、なし) を判断するには、UnityEngine.VR.WSA.WebCam.Mode をチェックします。

## 写真のキャプチャ

**名前空間:** UnityEngine.VR.WSA.WebCam

**型:** PhotoCapture

PhotoCapture 型により、フォト ビデオカメラを使用して静止画写真を撮影できるようになります。PhotoCapture を使用して写真を撮影する一般的なパターンは以下のとおりです。

1.  PhotoCapture オブジェクトを作成します。

2.  必要な設定で CameraParameters を作成します。

3.  StartPhotoModeAsync を使用してフォト モードを開始します。

4.  目的の写真を撮影します。

    -   (オプション) その写真を操作します。

5.  フォトモードを停止し、リソースをクリーンアップします。

## PhotoCapture の共通セットアップ

以下で説明する 3 つの用途はどれも、上記の最初の 3 つの手順から始めます。

まず、PhotoCapture オブジェクトを作成します。

```cs
PhotoCapture photoCaptureObject = null;
   void Start()
   {
       PhotoCapture.CreateAsync(false, OnPhotoCaptureCreated);
   }
```

次に、このオブジェクトを保存し、パラメーターを設定して、フォトモードを開始します。
```cs
void OnPhotoCaptureCreated(PhotoCapture captureObject)
   {
       photoCaptureObject = captureObject;

       Resolution cameraResolution = PhotoCapture.SupportedResolutions.OrderByDescending((res) => res.width * res.height).First();

       CameraParameters c = new CameraParameters();
       c.hologramOpacity = 0.0f;
       c.cameraResolutionWidth = cameraResolution.width;
       c.cameraResolutionHeight = cameraResolution.height;
       c.pixelFormat = CapturePixelFormat.BGRA32;

       captureObject.StartPhotoModeAsync(c, false, OnPhotoModeStarted);
   }
```

最後に、以下に示す同一のクリーンアップ コードも使用します。
```cs
void OnStoppedPhotoMode(PhotoCapture.PhotoCaptureResult result)
   {
       photoCaptureObject.Dispose();
       photoCaptureObject = null;
   }
```

これらの手順の後、キャプチャする写真の種類を選択できます。

## ファイルへの写真のキャプチャ

最も単純なのは、写真を直接ファイルにキャプチャする操作です。写真は JPG
または PNG として保存できます。

フォトモードが正常に開始されたら、写真を撮影するとその写真がディスクに保存されるようになります。
```cs
private void OnPhotoModeStarted(PhotoCapture.PhotoCaptureResult result)
   {
       if (result.success)
       {
           string filename = string.Format(@"CapturedImage{0}_n.jpg", Time.time);
           string filePath = System.IO.Path.Combine(Application.persistentDataPath, filename);

           photoCaptureObject.TakePhotoAsync(filePath, PhotoCaptureFileOutputFormat.JPG, OnCapturedPhotoToDisk);
       }
       else
       {
           Debug.LogError("Unable to start photo mode!");
       }
   }
```

写真をディスクにキャプチャしたら、フォトモードを終了し、オブジェクトをクリーンアップします。
```cs
void OnCapturedPhotoToDisk(PhotoCapture.PhotoCaptureResult result)
   {
       if (result.success)
       {
           Debug.Log("Saved Photo to disk!");
           photoCaptureObject.StopPhotoModeAsync(OnStoppedPhotoMode);
       }
       else
       {
           Debug.Log("Failed to save Photo to disk");
       }
   }
```

## Texture2D への写真のキャプチャ

データを Texture2D にキャプチャする手順は、ディスクにキャプチャするのとほぼ同じです。

セットアップは前述の手順に従います。

OnPhotoModeStarted で、フレームをメモリにキャプチャします。
```cs
private void OnPhotoModeStarted(PhotoCapture.PhotoCaptureResult result)
   {
       if (result.success)
       {
           photoCaptureObject.TakePhotoAsync(OnCapturedPhotoToMemory);
       }
       else
       {
           Debug.LogError("Unable to start photo mode!");
       }
   }
```

次に、結果をテクスチャに適用して、前述の共通クリーンアップコードを使用します。
```cs
void OnCapturedPhotoToMemory(PhotoCapture.PhotoCaptureResult result, PhotoCaptureFrame photoCaptureFrame)
   {
       if (result.success)
       {
           // Create our Texture2D for use and set the correct resolution
           Resolution cameraResolution = PhotoCapture.SupportedResolutions.OrderByDescending((res) => res.width * res.height).First();
           Texture2D targetTexture = new Texture2D(cameraResolution.width, cameraResolution.height);
           // Copy the raw image data into our target texture
           photoCaptureFrame.UploadImageDataToTexture(targetTexture);
           // Do as we wish with the texture such as apply it to a material, etc.
       }
       // Clean up
       photoCaptureObject.StopPhotoModeAsync(OnStoppedPhotoMode);
   }
```

## 写真のキャプチャとキャプチャしたバイト列そのものの操作

メモリ内のフレームのバイト列をそのまま操作するには、前述と同じセットアップ手順に従い、写真を Texture2D にキャプチャしたのと同じ OnPhotoModeStarted を使用します。異なるのは OnCapturedPhotoToMemory での処理です。ここでは、キャプチャしたバイト列そのものを取得して操作できます。

この例では List&lt;Color&gt; を作成します。List&lt;Color&gt; では細かい処理や、SetPixels() を使ったテクスチャへの適用が可能です。
```cs
void OnCapturedPhotoToMemory(PhotoCapture.PhotoCaptureResult result, PhotoCaptureFrame photoCaptureFrame)
   {
       if (result.success)
       {
           List<byte> imageBufferList = new List<byte>();
           // Copy the raw IMFMediaBuffer data into our empty byte list.
           photoCaptureFrame.CopyRawImageDataIntoBuffer(imageBufferList);

           // In this example, we captured the image using the BGRA32 format.
           // So our stride will be 4 since we have a byte for each rgba channel.
           // The raw image data will also be flipped so we access our pixel data
           // in the reverse order.
           int stride = 4;
           float denominator = 1.0f / 255.0f;
           List<Color> colorArray = new List<Color>();
           for (int i = imageBufferList.Count - 1; i >= 0; i -= stride)
           {
               float a = (int)(imageBufferList[i - 0]) * denominator;
               float r = (int)(imageBufferList[i - 1]) * denominator;
               float g = (int)(imageBufferList[i - 2]) * denominator;
               float b = (int)(imageBufferList[i - 3]) * denominator;

               colorArray.Add(new Color(r, g, b, a));
           }
           // Now we could do something with the array such as texture.SetPixels() or run image processing on the list
       }
       photoCaptureObject.StopPhotoModeAsync(OnStoppedPhotoMode);
   }
```

## ビデオのキャプチャ

**名前空間:** UnityEngine.VR.WSA.WebCam

**型:** VideoCapture

VideoCapture 関数は PhotoCapture とほぼ同じです。異なるのは、1 秒あたりのフレーム数 (FPS) の値を指定することと、.mp4 ファイル形式以外ではディスクに直接保存できないことの 2 点のみです。VideoCapture は以下の手順で使用します。

1.  VideoCapture オブジェクトを作成します。
2.  必要な設定で CameraParameters を作成します。
3.  StartVideoModeAsync を使用してビデオ モードを開始します。
4.  ビデオの録画を開始します。
5.  ビデオの録画を停止します。
6.  ビデオ モードを停止し、リソースをクリーンアップします。

まず、VideoCapture オブジェクトを作成します。

```cs
void Start ()
   {
       VideoCapture.CreateAsync(false, OnVideoCaptureCreated);
   }
```

次に、録画と開始に必要なパラメーターを設定します。
```cs
void OnVideoCaptureCreated (VideoCapture videoCapture)
   {
       if (videoCapture != null)
       {
           m_VideoCapture = videoCapture;

           Resolution cameraResolution = VideoCapture.SupportedResolutions.OrderByDescending((res) => res.width * res.height).First();
           float cameraFramerate = VideoCapture.GetSupportedFrameRatesForResolution(cameraResolution).OrderByDescending((fps) => fps).First();

           CameraParameters cameraParameters = new CameraParameters();
           cameraParameters.hologramOpacity = 0.0f;
           cameraParameters.frameRate = cameraFramerate;
           cameraParameters.cameraResolutionWidth = cameraResolution.width;
           cameraParameters.cameraResolutionHeight = cameraResolution.height;
           cameraParameters.pixelFormat = CapturePixelFormat.BGRA32;

           m_VideoCapture.StartVideoModeAsync(cameraParameters,
                                               VideoCapture.AudioState.None,
                                               OnStartedVideoCaptureMode);
       }
       else
       {
           Debug.LogError("Failed to create VideoCapture Instance!");
       }
   }
```

ビデオ モードが開始されたら、録画を始めます。
```cs
void OnStartedVideoCaptureMode(VideoCapture.VideoCaptureResult result)
   {
       if (result.success)
       {
           string filename = string.Format("MyVideo_{0}.mp4", Time.time);
           string filepath = System.IO.Path.Combine(Application.persistentDataPath, filename);

           m_VideoCapture.StartRecordingAsync(filepath, OnStartedRecordingVideo);
       }
   }
```

録画開始後に、UI または動作を更新して、録画を停止できるようにします。ここではログに記録するだけです。
```cs
void OnStartedRecordingVideo(VideoCapture.VideoCaptureResult result)
   {
       Debug.Log("Started Recording Video!");
       // We will stop the video from recording via other input such as a timer or a tap, etc.
   }
```

この後、録画を停止することになります。たとえば、タイマーを使ったり、ユーザーの入力を利用して停止します。
```cs
// The user has indicated to stop recording
   void StopRecordingVideo()
   {
       m_VideoCapture.StopRecordingAsync(OnStoppedRecordingVideo);
   }
```

録画が停止したら、ビデオモードを停止し、リソースをクリーンアップします。
```cs
void OnStoppedRecordingVideo(VideoCapture.VideoCaptureResult result)
   {
       Debug.Log("Stopped Recording Video!");
       m_VideoCapture.StopVideoModeAsync(OnStoppedVideoCaptureMode);
   }

   void OnStoppedVideoCaptureMode(VideoCapture.VideoCaptureResult result)
   {
       m_VideoCapture.Dispose();
       m_VideoCapture = null;
   }
```

## トラブルシューティング

-   利用できる解像度がない場合

    -   **Web カメラ機能** をプロジェクトで指定していることを確認します。

## 関連項目

-   [ロケータブルカメラ](https://developer.microsoft.com/ja-jp/windows/mixed-reality/locatable_camera)
