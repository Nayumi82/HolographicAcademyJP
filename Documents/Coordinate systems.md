**座標系**

ホログラフィック
アプリの大きな目的は、[*ホログラム*](https://developer.microsoft.com/ja-jp/windows/holographic/hologram)を仮想世界に配置し、実物のように見せ、実物のように音を出すことです。そのためには、仮想世界の中でユーザーにとって意味のある場所に、正確な位置と向きでホログラムを配置する必要があります。ホログラムの位置と向き、[*視線*](https://developer.microsoft.com/ja-jp/windows/holographic/gaze)の向きや[*手の位置*](https://developer.microsoft.com/ja-jp/windows/holographic/gestures)などのジオメトリの位置と向きを論理的に判断する際に、HoloLens
ではさまざまな座標系を使用してそのジオメトリを表現します。

**目次**

-   [*1
    空間座標系*](https://developer.microsoft.com/ja-jp/windows/holographic/coordinate_systems#spatial_coordinate_systems)

-   [*2
    静止座標系*](https://developer.microsoft.com/ja-jp/windows/holographic/coordinate_systems#stationary_frame_of_reference)

-   [*3
    空間アンカー*](https://developer.microsoft.com/ja-jp/windows/holographic/coordinate_systems#spatial_anchors)

    -   [*3･1
        すべてのシーンに単一の固定座標系を使用できない理由*](https://developer.microsoft.com/ja-jp/windows/holographic/coordinate_systems#why_a_single_rigid_coordinate_system_cannot_be_used_for_the_whole_scene)

    -   [*3.2 解決策:
        空間アンカー*](https://developer.microsoft.com/ja-jp/windows/holographic/coordinate_systems#the_solution:_spatial_anchors)

    -   [*3.3
        空間アンカーの保存*](https://developer.microsoft.com/ja-jp/windows/holographic/coordinate_systems#spatial_anchor_persistence)

    -   [*3.4
        空間アンカーの共有*](https://developer.microsoft.com/ja-jp/windows/holographic/coordinate_systems#spatial_anchor_sharing)

    -   [*3.5 ベスト
        プラクティス*](https://developer.microsoft.com/ja-jp/windows/holographic/coordinate_systems#best_practices)

-   [*4
    従属座標系*](https://developer.microsoft.com/ja-jp/windows/holographic/coordinate_systems#attached_frame_of_reference)

-   [*5 頭固定 (Head-locked)
    コンテンツ*](https://developer.microsoft.com/ja-jp/windows/holographic/coordinate_systems#head-locked_content)

-   [*6
    追跡エラーへの対処*](https://developer.microsoft.com/ja-jp/windows/holographic/coordinate_systems#handling_tracking_errors)

    -   [*6.1 センサー
        データの不足によりデバイスが追跡不能になる場合*](https://developer.microsoft.com/ja-jp/windows/holographic/coordinate_systems#device_cannot_track_due_to_insufficient_sensor_data)

    -   [*6.2
        環境の動的変化によりデバイスの追跡が不正確になる場合*](https://developer.microsoft.com/ja-jp/windows/holographic/coordinate_systems#device_tracks_incorrectly_due_to_dynamic_changes_in_the_environment)

    -   [*6.3
        時間と共に環境が大きく変化するためデバイスの追跡が不正確になる場合*](https://developer.microsoft.com/ja-jp/windows/holographic/coordinate_systems#device_tracks_incorrectly_because_the_environment_has_changed_significantly_over_time)

    -   [*6.4
        環境内の類似空間によりデバイスの追跡が不正確になる場合*](https://developer.microsoft.com/ja-jp/windows/holographic/coordinate_systems#device_tracks_incorrectly_due_to_identical_spaces_in_an_environment)

-   [*7
    関連項目*](https://developer.microsoft.com/ja-jp/windows/holographic/coordinate_systems#see_also)

**空間座標系**

すべての 3D グラフィック
アプリケーションは、仮想世界にレンダリングする対象物の位置と向きを論理的に決めるために、[*直交座標系*](https://en.wikipedia.org/wiki/Cartesian_coordinate_system)を使用します。このような座標系では、対象物の位置を決めるために
3 本の直交軸、X 軸、Y 軸、Z 軸を定めます。

HoloLens
では、アプリは仮想世界と現実世界の両方で座標系を定めることになります。Windows
Holographic では、現実世界の座標系を「**空間座標系**」と呼びます。

空間座標系では、座標の値をメートル単位で表します。つまり、X 軸、Y 軸、Z
軸のいずれかで 2 目盛り離れた位置にある対象物を HoloLens
でレンダリングするときは 2
メートル離して表示することになります。これにより、現実世界の基準で対象物をレンダリングするのが容易になります。

一般に、直交座標系は右手系または左手系のいずれかです。Windows
Holographic の空間座標系は常に右手系です。つまり、X
軸は右方向が正になり、Y 軸 (重力の向き) は上方向が正、Z
軸は仮想世界を見ている利用者の方向 (手前) が正になります。

どちらの種類の座標系も正の向きは X 軸では右方向、Y
軸では上方向になります。違いは Z
軸です。画面の手前に向かう方向が正になるか、奥に向かう方向が正になるかの違いです。このとき、右手を使うか左手を使うかで親指の向きが変わります。親指は、手前か、奥を指します。この親指が指す方向がその座標系での
Z 軸の正方向になります。

**静止座標系**

現実世界を表現する場合、デバイスは環境内を動き回るものとして、静止した状態で変わらない参照ポイントが必要になります。この目的でシンプルなアフォーダンスを提供する座標系を「静止座標系」と呼びます。静止座標系は、ユーザーの近くにある対象物の位置を現実世界と相対に、できる限り安定した状態を維持するように機能します。

Unity のようなゲーム
エンジンでは、静止座標系はエンジンという「世界の原点」を定義します。特定の世界の座標に配置する対象物は静止座標系を使用してその位置を定義しますが、現実世界でも同じ座標系を使用してその位置を判断します。通常、アプリでは開始時に
1
つの静止座標系を作成し、アプリのライフサイクル全体でこの座標系を使用します。

時間が経ち、ユーザーがいる環境について多くのことを学習するにつれ、現実世界のさまざまな地点との距離が、以前の認識よりも短くなったり、長くなったりする可能性があります。そのため、ユーザーが広い範囲を歩き回るにつれ座標系が調整され、結果として静止座標系に配置されたホログラムが元の位置からずれて見えることもあります。

**空間アンカー**

現実世界についての認識を深めても、こうしたずれを防ぎ、ホログラムが現実世界の特定の位置に正確にとどまり続けるように、ホログラムの配置に「**空間アンカー**」を利用できます。

空間アンカーとは、時間が経っても座標系が追跡を続ける重要な点を表します。各アンカーはそれぞれ座標系を持ちます。この座標系は、必要に応じて他の空間アンカーや他の座標系と相対に調整され、アンカーを利用するホログラムが正しい位置にとどまるようにします。

空間アンカーの座標系でホログラムをレンダリングすることで、任意の時点で最も正確な位置にホログラムを配置できます。これにより、座標系が現実世界に合わせて絶えず本来の位置にホログラムを動かすことになるため、時間と共にホログラムの位置をわずかに調整するという手間がかかります。

**すべてのシーンに単一の固定座標系を使用できない理由**

現状、ゲーム、データ表示アプリ、仮想現実アプリを作成する一般的な方法として、他のすべての座標系が信頼して基準にする絶対的なワールド座標系を
1 つ確立します。そのような環境では、その世界の任意の 2
つの対象物の関係を定める安定した変換が必ず見つかります。これらの対象物を動かさなければ、それらの相対変換は常に同じになります。この種のグローバルな座標系は、事前にすべてのジオメトリを把握できる、純粋な仮想世界をレンダリングするのであれば適切に機能します。

一方、複合現実デバイスである HoloLens
は、センサーの主導により世界を動的に認識します。つまり、作成した空間アンカーの知識が、時間と共に絶えず調整されます。そのため、アプリでは、時間と共に空間アンカーの関係が相互に変化するように、作成する空間アンカーを準備する必要があります。

たとえば、デバイスが、現在 2 つの場所の間に 4
メートルの距離があると認識していて、その後認識の精度が上がり、その距離が
3.9 メートルであると学習するとします。2
つのホログラムを単一の固定座標系で 4
メートル離れて配置すると、現実世界よりも常に 0.1
メートル離れて表示されることになります。

**解決策: 空間アンカー**

HoloLensでこの問題を解決するには、仮想世界の中でユーザーがホログラムを配置する重要な地点をマークする空間アンカーを作成します。デバイスが仮想世界を学習するにつれ、これらの空間アンカーが必要に応じて互いの相対位置を調整し、現実世界に合わせて配置された場所に正確に留まるようにできます。空間アンカー近くの座標系内にホログラムを配置することで、そのホログラムが最適な安定性を維持するようになります。

空間アンカーを利用する座標系と静止座標系の大きな違いは、空間アンカーどうしの相対位置が絶えず調整される点にあります。

-   静止座標系に配置されたホログラムは、常に相互の固定関係を保持します。ただし、ユーザーの隣にホログラムが安定して表示されるように、世界と相対に静止座標系が移動します。

-   ある空間アンカーを使用して配置されたホログラムは、別の空間アンカーを使用して配置されたホログラムと相対に移動します。これにより、たとえば、あるアンカーを左に動かし、別のアンカーを右に動かす調整が必要でも、Windows
    は各空間アンカーの位置の認識を改善できます。

静止座標系ではユーザーの近くに安定させようと最適化するのに対し、空間アンカーは原点の近くに安定させようとします。これにより、ホログラムは時間が経っても正確な場所にとどまり続けることができますが、空間アンカーの原点から遠く離れた場所にレンダリングされるホログラムは、深刻なレバーアーム効果の影響を受けやすくなります。これは、空間アンカーの位置と向きの小さな更新が、そのアンカーからの距離に比例して拡大するためです。これまでの経験による対策として、空間アンカーの座標系に基づいてレンダリングするのは、その空間アンカーの原点から約
3 メートル以内のものにします。

**空間アンカーの保存**

空間アンカーにより、アプリが停止したりデバイスがシャットダウンしても、重要な場所を記憶しておくことができます。

アプリで作成した空間アンカーをアプリの「**空間アンカー
ストア**」に永続化することで、空間アンカーをディスクに保存し、後から読み込むことができるようになります。アンカーを保存または読み込む際、アプリにとって意味のある文字列キーを指定して、そのアンカーを特定します。このキーはアンカーのファイル名と考えます。ユーザーがその場所に配置した
3D
モデルなど、他のデータをそのアンカーに関連付ける場合は、そのデータをアプリのローカル
ストレージに保存して、選択したキーに関連付けます。

アンカーをストアに保存することで、ユーザーは個々のホログラムを配置したり、アプリがさまざまなホログラムを配置するワークスペースを配置することができます。その後、アプリを何度も使用するにつれて、ユーザーが想定する場所にホログラムが表示されるようになります。

**空間アンカーの共有**

アプリでは、空間アンカーを他のデバイスと共有することもできます。空間アンカーを別の
HoloLens に転送し、その空間アンカーを取り巻く環境の認識や関連センサー
データも一緒に提供することで、両方のデバイスが同じ場所を特定できるようになります。共有空間アンカーを使用して各デバイスでホログラムをレンダリングすることで、両デバイスのユーザーは現実世界の同じ場所にホログラムを表示することができます。

**ベスト プラクティス**

空間アンカーの最適な使用法、および空間アンカーの代わりに静止座標系を使用する際のガイドラインについては、[*空間アンカーのベスト
プラクティス*](https://developer.microsoft.com/ja-jp/windows/holographic/spatial_anchors)
ページを確認してください。

**従属座標系**

ユーザーに追従し、常に特定のユーザーの頭から一定距離を保って浮遊するよう設計されるホログラムもあります。こうしたホログラムは「従属座標系」に配置できます。従属座標系は、ユーザーが歩きまわると一緒に移動します。従属座標系は向きが決まっている点に注意が必要です、この向きは、最初に従属座標系を作成するときに定義します。この座標系は、ユーザーが頭や体の向きを変えても回転しません。これにより、ユーザーの動きにホログラムが追従しても、その座標系内に配置されたさまざまなホログラムを快適に見回すことができます。ユーザーの動きに合わせて表示されるコンテンツを「体固定
(body-locked)」コンテンツと呼びます。

デバイスが仮想世界のどこにいるかを把握できなくても、従属座標系を使用するホログラムだけはレンダリングすることができます。これをフォールバック
UI
として表示して、デバイスが仮想世界の中で自身の位置を検出できないことをユーザーに伝えることができます。すべてのアプリにこのようなフォールバックを含め、[*holographic
shell*](https://developer.microsoft.com/ja-jp/windows/holographic/navigating_the_windows_mixed_reality_home)
に表示されるような UI を使ってユーザーを支援するようにします。 .

**頭固定 (Head-locked) コンテンツ**

ディスプレイ (HUD など)
の固定位置に表示される頭固定コンテンツのレンダリングはお勧めできません。一般に、頭固定コンテンツはユーザーを不快にし、現実世界の自然な現象のようには感じられません。

頭固定コンテンツは、通常、ユーザーに追従するホログラムに置き変えるか、仮想世界自体に固定します。たとえば、[*カーソル*](https://developer.microsoft.com/ja-jp/windows/holographic/cursors)は、ユーザーの視線の先にある対象物の位置と距離に応じて自然な大きさにして、仮想世界に配置します。

**追跡エラーへの対処**

環境によっては、デバイスが仮想世界の中での自身の位置を正確に認識できない場合があります。その結果、ホログラムが前と同じ位置に表示されなかったり、不適切な場所に表示されることになります。ここでは、このような現象が発生する状況、それがユーザー
エクスペリエンスに及ぼす影響、こうした状況から回復するためのヒントについて考えます。

**センサー データの不足によりデバイスが追跡不能になる場合**

デバイスのセンサーがデバイスの場所を特定できなくなることがあります。部屋が暗い場合、センサーが髪や手で隠れている場合、周囲に十分なテクスチャがない場合にこうした現象が起こります。

このような状況が発生すると、デバイスは自身の場所を正確に追跡できず、仮想世界固定のホログラムをレンダリングできなくなります。そうなると、空間アンカーや静止座標系とデバイスとの相対位置を判断することはできませんが、従属座標系内の体固定コンテンツはレンダリングできます。

アプリでは、こうした体固定コンテンツをフォールバックとして表示して、センサーが隠れていることや、もっと明るくすることを求めるヒントを示し、位置の追跡を回復する方法をユーザーに通知します。

**環境の動的変化によりデバイスの追跡が不正確になる場合**

環境に多くの動的変化が生じると、デバイスは適切に追跡できなくなることがあります。たとえば、部屋の中をたくさんの人が歩き回っている場合です。このような場合、デバイスが動的環境の中で自身を追跡しようとしても、ホログラムが大きく移動したり、ずれて見えることがあります。このような状況になったら、動的変化の少ない環境でデバイスを使用することをお勧めします。

**時間と共に環境が大きく変化するためデバイスの追跡が不正確になる場合**

家具の配置が変わったり、壁掛けが移動するなど、環境に大きな変化が起きた後にデバイスを使うと、一部のホログラムが本来の場所から移動して表示されることがあります。この新しい空間でユーザーが動き回ると、ホログラムが最初に表示された位置から大きく移動することもあります。これは、デバイスの空間認識が保持されなくなり、デバイスがホログラムの位置を調整しながら環境の再認識を試みるためです。このような状況では、最初のホログラムを削除することをお勧めします。それでも、新しく配置したホログラムがずれたり、大きく移動するように見える場合は、ユーザーにデバイスで現在の空間を削除するよう依頼します。それには、\[Settings
(設定)\]、\[System (システム)\]、\[Spaces (空間)\]
の順に移動します。空間を削除すると多くのホログラムが失われるため注意が必要です。

**環境内の類似空間によりデバイスの追跡が不正確になる場合**

家や空間によく似た 2
つの場所が存在することがあります。たとえば、家の中によく似た部屋が 2
つあり、デバイスの視野には部屋のコーナーも、大きなポスターも同じように見える場合があります。このような状況では、デバイスはよく似た対象物に混乱し、内部表現としては同じものだと見なす可能性があります。これにより、ホログラムが別の場所に表示されるかもしれません。デバイス内での環境の内部表現が損なわれることにより、追跡できなくなり始めることはよくあります。このような場合は、\[Settings
(設定)\]、\[System (システム)\]、\[Spaces (空間)\]
の順に移動して、現在の空間を削除することをお勧めします。空間を削除すると、多くのホログラムが失われるため注意が必要です。空間を削除すれば、デバイスは環境固有の領域を適切に追跡できるようになります。ただし、デバイスがよく似た領域で再び混乱を生じると、問題が再発する可能性があります。

  --- -------------------------------------------------------------------------------------------------------------------------
  ⚠   **このセクションには、画像が必要です。**追跡を失ったときに表示される Shell UI のスクリーン ショットを追加してください。
  --- -------------------------------------------------------------------------------------------------------------------------

**関連項目**

-   [*空間アンカー*](https://developer.microsoft.com/ja-jp/windows/holographic/spatial_anchors)

-   [*ホログラフィック共有エクスペリエンス*](https://developer.microsoft.com/ja-jp/windows/holographic/shared_holographic_experiences)

-   [*Unity の World
    アンカー*](https://developer.microsoft.com/ja-jp/windows/holographic/world_anchor_in_unity)

-   [*DirectX
    の座標系*](https://developer.microsoft.com/ja-jp/windows/holographic/coordinate_systems_in_directx)

-   [*DirectX
    の共有空間アンカー*](https://developer.microsoft.com/ja-jp/windows/holographic/shared_spatial_anchors_in_directx)

-   [*ケース スタディ –
    現実世界で穴を通して見る*](https://developer.microsoft.com/ja-jp/windows/holographic/case_study_-_looking_through_holes_in_your_reality)

