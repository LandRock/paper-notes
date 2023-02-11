![image.png](https://cdn.nlark.com/yuque/0/2023/png/34409005/1675749497579-0a03a35f-ba58-48f5-a12e-43b02935ddd4.png#averageHue=%237a6c53&clientId=u351badaf-db3d-4&from=paste&height=612&id=ue830e699&name=image.png&originHeight=765&originWidth=2140&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1175582&status=done&style=none&taskId=u219c0c02-3d7a-4326-b044-cadae65230b&title=&width=1712)
# Abstract
本文提出了第一个实时 3D 眼睛注视捕获方法，该方法使用单个 RGB 相机同时捕获 3D 眼睛注视、头部姿势和面部表情变形的协调运动。我们的主要想法是用高效的 3D 眼睛注视跟踪器来补充实时 3D 面部表演捕捉系统。我们通过自动检测每一帧的重要 2D 面部特征来开始这个过程。然后使用检测到的面部特征使用多线性表情变形模型重建 3D 头部姿势和大规模面部变形。接下来，我们介绍了一种新的独立于用户的分类方法，用于在每一帧中提取虹膜和瞳孔像素。我们在 Maximum A Posterior (MAP) 框架中制定了 3D 眼睛注视跟踪器，该框架依次推断出每帧 3D 眼睛注视的最可能状态。当眨眼发生时，眼睛注视跟踪器可能会失败。我们进一步引入了一种高效的闭眼检测器，以提高眼睛注视跟踪器的鲁棒性和准确性。我们已经在实时视频流和互联网视频上测试了我们的系统，证明了它在各种不受控制的光照条件下的准确性和稳健性，并克服了种族、性别、体型、姿势和表情在个体之间的显着差异。
# 1 Introduction
面部动画是许多应用程序的重要组成部分，例如电影、视频游戏和虚拟环境。迄今为止，最流行和最成功的创建虚拟面孔的方法之一通常是捕捉真人的面部表情。面部表演捕捉问题的理想解决方案是使用标准摄像机以 3D 方式捕捉现场表演。单个摄像机的最低要求特别有吸引力，因为它提供最低的成本、简化的设置以及对遗留源和不受控制的视频（例如 Internet 视频）的潜在使用。
计算机图形学和视觉的最新进展已经允许开发一系列令人印象深刻的 3D 面部表现捕捉方法，包括在线 [Cao 等人。 2014a;曹等。 2015] 和离线系统 [Garrido 等人。 2013;施等。 2014]，使用单个 RGB 相机。值得注意的是，Cao 和他的同事 [2015] 提出了第一个实时面部表现捕捉系统，用于捕捉 3D 头部姿势、大规模面部变形和中等规模的面部细节，如表情皱纹。然而，所有以前的面部捕捉系统都缺乏捕捉面部表现不可或缺的组成部分的能力：眼睛注视。正如 Ruhland 等人所指出的。 [2014]，拉丁谚语说：“脸是心灵的画像；眼睛，它的告密者。”眼睛是传达情感信息的核心，因为我们能够通过观察他人的眼睛来解读他人的意图和感受。逼真的 3D 虚拟眼睛的动画和控制仍然具有挑战性，因为眼睛注视非常微妙，人眼很容易辨别任何不自然的注视，并且可能暗示观察者有错误的意图。
本文介绍了第一个实时 3D 眼睛注视捕获方法，该方法使用单个 RGB 相机同时跟踪 3D 眼睛注视、头部姿势和大规模面部变形（见图 1）。我们通过自动检测重要的面部特征（例如每个输入帧的鼻尖）来开始该过程。然后使用检测到的面部特征使用多线性表情变形模型重建 3D 头部姿势和大规模面部变形。接下来，我们训练一个与用户无关的虹膜和瞳孔像素分类器基于随机森林，并用它来提取每一帧中的虹膜和瞳孔像素。我们在 MAP 框架中制定了眼睛注视跟踪器，它使用重建的 3D 头部姿势和当前帧的分类虹膜和瞳孔像素，以及从前一帧。当眨眼发生时，眼睛注视跟踪器通常会失败。这一挑战促使我们开发一种新颖的闭眼检测器，以进一步提高我们的眼睛注视跟踪器的稳健性和准确性。
最终的面部表演捕捉系统是强大的全自动的，允许使用单个 RGB 相机同时捕捉 3D 头部姿势、大规模面部表情变形和 3D 眼睛注视。我们已经在实时视频流和互联网视频上测试了我们的系统，证明了它在各种不受控制的光照条件下的准确性和稳健性，并克服了种族、性别、体型、姿势和表情在个体之间的显着差异。我们通过与人类受试者注释的地面实况数据进行比较来评估捕获的眼睛注视的质量。此外，我们评估了 3D 凝视跟踪器关键组件的重要性，并通过与替代系统的比较表明我们的系统达到了最先进的精度。最后，我们展示了我们的表演捕捉系统在基于表演的面部动画、实时注视数据捕捉和眼睛注视可视化中的应用。
## 1.1 Contributions
我们的系统通过以下技术贡献成为可能：

- 首先也是最重要的是，第一个实时 3D 眼睛注视捕捉系统，可与任何使用 RGB 图像的面部表演捕捉系统相辅相成。
- 一种基于随机森林的新型用户独立虹膜和瞳孔像素分类器。
- 一种高效的眼睛注视跟踪器，应用重要性采样来推断 MAP 框架中最可能的眼睛注视状态。
- 一种新的闭眼检测器，可显着提高我们的眼睛注视跟踪器的稳健性和准确性。
# 2 Background
我们的实时面部表现系统使用单眼 RGB 相机自动跟踪 3D 眼睛注视、3D 头部姿势和面部表情变形。因此，我们将讨论重点放在为获取 3D 面部表演和凝视运动而开发的方法和系统上。
## 2.1 Facial Performance Capture
最近，Cao 和同事扩展了级联形状回归的思想用于 3D 面部捕捉。他们的第一个系统（Displaced dynamic expression regression for real-time facial tracking and animation.）训练了一个特定于用户的 3D 形状回归器，并使用它在运行时直接跟踪 2D 图像序列的 3D 面部表情变形。接下来，Cao 及其同事 Facewarehouse: a 3d facial expression database for visual computing. 提出了一种独立于用户的位移动态表达式回归，可在跟踪过程中自适应地细化相机矩阵和用户身份。最近，Cao 及其同事Realtime high-fidelity facial performance capture. 通过添加本地用户特定的细节回归器进一步将想法扩展到实时高保真面部捕捉。除了这些在线技术之外，还有离线系统将大规模表情变形跟踪与 shapefrom-shading 相结合，以捕捉详细和动态的 3D 面部几何形状 [Garrido et al. 2013;施等。 2014]。
我们的工作通过在面部捕捉中添加实时 3D 眼睛注视跟踪器来增强面部表现捕捉。这种增强使我们能够捕捉 3D 头部姿势、面部表情和眼睛注视之间的协调运动，这是以前任何工作都没有展示过的能力。值得一提的是，我们的框架非常灵活，我们的眼睛注视跟踪器可以与任何使用 RGB 图像的 3D 面部捕捉系统集成（例如，[Beeler et al. 2011; Valgaerts et al. 2012; Bouaziz et al. 2013; Li et al. 2013; Shi et al. 2014; Cao et al. 2015]）以捕捉 3D 头部姿势、面部变形和眼睛注视之间的协调运动。
## 2.2 Eye Gaze Tracking
几十年来，眼睛注视跟踪和眼睛检测一直是人机交互和计算机视觉领域的一个活跃研究课题。以前的方法主要集中在二维注视检测和跟踪上，可以分为两类：基于红外照明的方法和基于图像的方法。基于主动红外照明的方法利用在红外照明下称为角膜反射的光谱（反射）特性来有效检测虹膜和瞳孔像素，而基于图像的方法旨在根据人类的形状和/或外观检测或跟踪眼睛注视眼睛。
基于主动 IR 的方法（例如 [Morimoto 和 Flickner 2000]）是最成功的眼睛注视捕捉方法之一。由于简单有效，几乎所有商业眼动仪（例如，[Anon，Applied science laboratories 2015；Lc Technologies 2015；Tobii Technologies 2015]）都基于此技术。然而，基于 IR 的方法具有侵入性，因为它们需要用户佩戴特殊眼镜或设置专用 IR 设备进行注视捕捉。此外，与我们的方法不同，它们通常只关注 2D 眼睛注视捕捉。因此，它们不能灵活地捕捉不受控制的视频中的 3D 眼睛注视。
基于图像的传统方法可以进一步分为三类：模板匹配[Chau and Betke 2005；柯克伦等人。 2012]，基于外观的方法 [Huang 和 Wechsler 1999； Huang 和 Mariani 2000] 以及基于特征的方法 [Kawato 和 Ohya 2000；田等。 2000]。然而，那些方法通常不够稳健，无法处理光照、主体、头部姿势和面部表情的变化。最近的努力主要集中在应用级联形状回归 [Cao et al。 2012] 或用于瞳孔中心检测的深度学习方法。通过在眼睛瞳孔中心添加两个额外的标志，这些方法可以检测单个图像中的面部特征位置和瞳孔中心。值得注意的是，由旷视科技公司开发的 Face++ tracker [2015] 利用深度神经网络模型检测和跟踪 2D 地标，并在 2D 面部特征和瞳孔中心检测方面实现了最先进的性能。我们的眼睛注视检测和跟踪方法是不同的，因为我们将虹膜和瞳孔像素分类、眼睛闭合检测、虹膜和瞳孔区域的边缘图以及 3D 头部姿势结合起来进行 3D 眼睛注视跟踪。第 7.2 节显示我们的眼睛注视检测实现了更准确的结果。我们的目标也不同，因为我们专注于使用 3D 眼睛注视进行 3D 面部表现捕捉，而不是 2D 面部特征检测和瞳孔中心检测。
我们的工作与最近基于外观的注视估计相关[Sugano et al。 2014;伍德等。 2015;张等。 2015]，它直接从输入眼睛图像和 3D 头部姿势到眼睛注视学习回归函数。简而言之，这些系统首先使用通用 3D 面部模型和从输入图像中检测到的六个 2D 面部特征（包括左右嘴角和左右眼角的四个角）估计 3D 头部姿势，然后应用深度神经网络网络直接从 2D 眼睛图像和 3D 头部姿势到眼睛注视学习回归函数。我们的系统在以下几个方面与他们的不同。首先，与他们的系统不同，我们的系统非常灵活，可以与任何现有的 3D 面部表演捕捉系统集成，因为它不需要任何离线相机校准过程来进行注视捕捉。其次，我们在系统中引入了闭眼检测器，从而显着提高了眼睛注视跟踪器的稳健性和准确性。最后，与他们旨在从单个图像估计眼睛注视的系统不同，我们的注视跟踪专注于 3D 面部性能捕获，它同时从单眼 RGB 序列跟踪 3D 头部姿势、面部变形和眼睛注视。
# 3 Overview
我们的目标是构建一个实时面部表情捕捉系统，使用单目 RGB 相机稳健准确地跟踪 3D 眼睛注视、头部姿势和面部表情变形。这个问题很有挑战性，因为头部姿势、面部表情变形化和眼睛注视运动经常耦合在一起。准确估计 3D 眼睛注视通常需要准确估计双眼周围的 3D 头部姿势和面部变形。另一方面，准确检测眼睛注视和闭眼/睁眼可以进一步提高双眼周围面部表情变形的重建精度。此外，从 3D 到 2D 的投影中深度信息丢失导致的模糊性以及未知的相机参数和照明条件进一步使问题复杂化。为了应对这一挑战，我们提出了一种端到端的面部表现系统，该系统使用单个 RGB 相机同时跟踪 3D 头部姿势、眼睛注视和面部表情变形。整个系统由五个主要部分组成，总结如下（见图 2）。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34409005/1676106951523-15ee5212-3fb1-4c8a-88d9-be6a8a2b918d.png#averageHue=%23eeedeb&clientId=u0138e13d-8cd3-4&from=paste&height=641&id=uf1d3ce6c&name=image.png&originHeight=801&originWidth=2033&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=318189&status=done&style=none&taskId=ubafa1e73-76c8-415b-bdd7-15aafbc42b7&title=&width=1626.4)
3D facial reconstruction.我们通过自动检测和跟踪重要的面部特征（例如单眼视频序列中的鼻尖）来开始该过程。我们引入了一种数据驱动的 3D 面部重建技术，使用多线性表情变形模型重建 3D 头部姿势和大规模表情变形。
Pixel classification for iris and pupil.我们引入了一种新颖的与用户无关的像素分类器，以自动注释眼睛区域中的虹膜和瞳孔像素，该像素由眼睛区域中检测到的面部标志所界定。我们通过对分类的虹膜和瞳孔像素应用均值漂移算法 [Comaniciu 和 Meer 2002] 进一步获得瞳孔中心的 2D 位置。我们讨论了如何提取虹膜的外部轮廓（即角膜缘）以进一步提高我们的注视跟踪器的鲁棒性和准确性。
Automatic eyeball calibration.在整个视频序列中跟踪 3D 眼睛注视不仅需要对 3D 眼球的几何形状进行建模，还需要估计眼球的位置以及虹膜和瞳孔区域的大小。我们用特定半径（12.5 毫米）的球体来近似眼球的几何形状，这对应于成人眼球的平均半径。我们引入了眼球校准步骤，该步骤在每次捕获开始时自动完成，以估计眼球的 3D 位置以及虹膜和瞳孔区域的大小。
Eye gaze tracking.我们根据每个瞳孔中心在眼球球体表面的位置来表示 3D 眼睛注视状态。我们根据检测到的 2D 瞳孔中心、虹膜外轮廓和估计的 3D 头部姿势依次更新眼睛注视状态。我们在最大后验概率 (MAP) 框架中制定问题，并应用重要性采样来推断最可能的眼睛注视状态。
Eye close detection.在实践中，我们观察到眼睛注视跟踪算法经常在眨眼时失败。这一挑战促使我们推出一种新颖的闭眼检测器，以自动检测眼睛是睁开还是闭上。一旦眼睛闭合，我们关闭虹膜和瞳孔像素分类器和注视跟踪，并直接使用前一帧的结果预测眼睛注视的状态。此外，我们还讨论了如何使用嵌入在训练数据中的眼睛注视约束来进一步提高系统的准确性和鲁棒性。
# 4 2D Facial Feature Tracking and 3D Face Reconstruction
本节讨论如何从 RGB 视频序列重建 3D 头部姿势和面部表情变形，这对我们的 3D 凝视跟踪器至关重要。我们从 2D 面部特征检测/跟踪开始（图 3 (b)），它建立在基于局部二元特征 (LBF) 的回归 [Ren et al. 2014]。接下来，我们使用跟踪的 2D 特征重建 3D 头部姿势和大规模表情变形（图 3（c））。
## 4.1 2D Facial Feature Detection/Tracking
此步骤旨在检测和跟踪单目视频中的二维面部特征。该任务是通过基于局部二元特征 (LBF) 的回归来实现的，该回归已被证明优于级联回归 [Cao 等人。 2012] 在准确性和效率方面。局部二元特征是由在每个地标处获得的一维二元特征组装而成的长一维向量。给定这个向量，面部形状 S（即 2D 面部特征位置的集合）通过逐步估计形状增量 ΔS 逐步细化。使用输入图像对阶段 t 的形状增量 ΔSt 进行回归，并使用随机森林提取局部二元特征（等式 1）。
这个方法应该过时了，可以换一个方法。
## 4.2 3D Facial Performance Reconstruction
我们现在描述如何从跟踪的 2D 位置重建 3D 面部变形和头部姿势。类似于 Shi 等人。 [2014]，我们使用多线性模型（等式 4）表示 3D 面部模型 [Vlasic 等人。 2005年；曹等。 2013]，并在优化框架中制定问题。
我们使用多线性模型 [Vlasic 等人] 表示 3D 面部模型。 2005年；曹等。 2013]。具体来说，我们使用两个低维向量来描述 3D 人脸，分别控制 3D 人脸的身份和表情：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34409005/1676116716937-cccd8dc7-c13c-4cc1-9a04-1bdaaefdb8bd.png#averageHue=%23f8f8f8&clientId=u0138e13d-8cd3-4&from=paste&height=71&id=u96eab5b9&name=image.png&originHeight=89&originWidth=819&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=7429&status=done&style=none&taskId=u08c25f47-a582-4b9a-a0c2-0aa45810f90&title=&width=655.2)
其中 M 表示未知主体的大规模面部几何形状，R 和 T 表示主体的全局旋转和平移，Cr 是减少的核心张量，mid 和 mexp 分别是身份和表情参数。我们的多线性模型是从 FaceWarehouse [Cao 等人构建的。 2014b]，其中包含对应于 150 个身份和 47 个面部表情的面部网格。在我们的实验中，身份和表达参数的维数设置为 50 和 25。
（其实还是3DMM方法）
通过假设一个理想的针孔相机模型，图像空间中的投影二维特征可以表示为：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34409005/1676116796177-d02ac3ff-372e-4ec5-ba53-94f00202a057.png#averageHue=%23f6f6f6&clientId=u0138e13d-8cd3-4&from=paste&height=74&id=u57d2064a&name=image.png&originHeight=93&originWidth=862&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=9178&status=done&style=none&taskId=ua296bedf-9a97-4f9a-b6f5-6365ab18a5a&title=&width=689.6)
其中 Q = [f 0 u; 0 f; 0 0 1] 是理想的针孔投影矩阵，(R, T) 是 3D 旋转和平移，f 是焦距，(u, v) 是主点。
这里的目标是最小化检测到的 2D 特征与假设人脸模型的投影 2D 特征之间的差异。类似于 Shi 等人。 [2014]，还强加了表达和姿势的额外先验和平滑度。请注意，身份权重和焦距仅在视频开始时进行估计，然后在其余帧中固定。我们在[Cao et al。 2013] 找到最佳焦距。主点（u，v）设置在图像的中心。对于其余帧，目标函数如下。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34409005/1676116868089-932630d9-240b-4eb0-b2f3-83a7a8787699.png#averageHue=%23f5f5f5&clientId=u0138e13d-8cd3-4&from=paste&height=76&id=uf19b7733&name=image.png&originHeight=95&originWidth=904&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=11131&status=done&style=none&taskId=ue7f5aa8a-dc43-438e-a73c-da0f287e437&title=&width=723.2)
（估计类似BFM模型的参数）
其中第一项是特征项，用于衡量重建的面部几何形状与整个序列中观察到的面部特征的匹配程度。第二项是用于正则化表达式参数的先验项，它被表述为多元高斯分布。第三项和第四项是平滑度项，用于惩罚表情和姿势随时间的突然变化。在我们所有的实验中，w1、w2 和 w3 分别设置为 0.00001、100 和 10。
姿势平滑度项限制帧之间的大旋转和平移变化：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34409005/1676116922219-34468e8e-7fe7-4d84-ba58-854b4b3a5ba8.png#averageHue=%23f6f6f6&clientId=u0138e13d-8cd3-4&from=paste&height=66&id=u170b6f3d&name=image.png&originHeight=83&originWidth=743&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=6833&status=done&style=none&taskId=uc6ef7b8d-3a2f-4ae3-b3c0-d2d77fe6423&title=&width=594.4)
其中 w4 和 w5 分别设置为 1 和 0.1。
# 5 User-independent Iris and Pupil Pixel Classifier
论文中用的随机森林的方法来进行语义分割。目的就是为了找出虹膜的像素和瞳孔中心位置。具体实现可以采用其他方法。
# 6 Model Based 3D Eye Gaze Tracking
本节介绍我们关于如何从重建的 3D 头部姿势、面部变形（第 4 节）和提取的 2D 瞳孔中心和边缘图（第 5 节）跟踪 3D 眼睛注视的想法。我们首先校准眼球中心以及虹膜和瞳孔区域的大小，这定义了用户特定的眼睛模型。然后我们逐帧跟踪瞳孔中心。我们在最大后验概率 (MAP) 框架中制定问题，并应用重要性采样技术来推断眼睛注视的最可能状态。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34409005/1676117126566-5e295ed7-296b-49b5-b905-c8e3c5b031a0.png#averageHue=%23f0eeec&clientId=u0138e13d-8cd3-4&from=paste&height=783&id=ub84cda62&name=image.png&originHeight=979&originWidth=1041&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=227248&status=done&style=none&taskId=u4dbc272b-af2e-4870-af09-a717acd2e2c&title=&width=832.8)
## 6.1 Representation
我们将眼睛注视状态 V 表示为：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34409005/1676117202437-a40b82be-fe23-4319-99e1-bfa69c80c5b1.png#averageHue=%23fbfbfb&clientId=u0138e13d-8cd3-4&from=paste&height=71&id=uab1cac94&name=image.png&originHeight=89&originWidth=630&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=4147&status=done&style=none&taskId=u1690c250-74fc-48dc-aa86-0b390c8e296&title=&width=504)
其中 P = (px, py, pz) 是人脸模型空间的眼球中心，s 是虹膜和瞳孔区域的大小（半径），(φ, θ) 是瞳孔中心，表示为球面坐标在眼球表面（图 8）。具体来说，眼球中心、虹膜和瞳孔大小定义了一个个性化的眼球模型，它们需要针对每个人进行校准。对于给定的 (P, s)，瞳孔中心 (φ, θ) 然后确定每个帧的 3D 眼睛注视运动。
## 6.2 Eyeball Calibration
我们的第一步是使用重建的 3D 头部姿势和面部变形以及提取的虹膜和瞳孔区域的边缘图来校准眼球中心和虹膜和瞳孔区域的大小。请注意，此步骤对每次捕获仅执行一次，因为每个人的眼球中心以及虹膜和瞳孔半径都是恒定的。
为了获得眼球中心，可以根据图像外观和重建的3D面部模型拟合一个球体。然而，由于只有部分眼球可见，估计可能不可靠。为了简单和更好的鲁棒性，我们使用固定的眼球半径（12.5mm），这是平均成人眼球半径。然后可以将眼球中心计算为面部模型上预选眼睑顶点的平均值加上将半径距离移向 3D 面部的主要方向（在我们的例子中为 z 方向）的 3D 偏移量。这使我们能够使用其中心和半径来定义眼球模型。
我们现在描述如何估计虹膜和瞳孔区域的大小。我们假设主体在起始帧中完全睁开眼睛看着相机，并且前 20 个有效帧用于校准。基于提取的虹膜和瞳孔区域的边缘图（见第 5 节），我们首先执行霍夫变换以拟合每个虹膜和瞳孔区域的圆。然后我们使用 3D 头部姿势将 2D 圆回投影到 3D 眼球模型，并在模型空间上获得相应的虹膜和瞳孔半径。最后，我们对所有半径进行平均，得到虹膜和瞳孔区域的最终大小。
## 6.3 3D Eye Gaze Tracking
有了已知的眼球中心、虹膜和瞳孔半径，我们的下一步是在每一帧跟踪剩余的注视状态，即 3D 瞳孔中心 (φ, θ)。眼睛注视运动具有复杂的模式，简单的时间跟踪很容易受到误差累积的影响。另一方面，从当前帧中提取的观察结果（2D 瞳孔中心和边缘图）虽然稳健，但缺乏详细的准确性。为了应对这一挑战，我们建议将提取的 2D 瞳孔中心、边缘信息和时间相干性组合到 MAP 框架中（等式 11）。然后我们通过重要性采样解决最可能的眼睛注视状态。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34409005/1676117346498-8e4eb185-1930-45e4-8b14-8a57dec27adf.png#averageHue=%23f9f9f9&clientId=u0138e13d-8cd3-4&from=paste&height=101&id=ud8a2ecc0&name=image.png&originHeight=126&originWidth=809&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=8340&status=done&style=none&taskId=u619a9873-5a77-4f87-a262-5d611626cde&title=&width=647.2)
其中 xt 是当前帧 t 的状态，Ht 是时间 t 中的头部方向，ot 是当前观察。使用贝叶斯规则，我们得到
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34409005/1676117383281-3cd6fc1f-7b55-4466-8ae1-175a28177808.png#averageHue=%23f0f0f0&clientId=u0138e13d-8cd3-4&from=paste&height=110&id=u0eea3587&name=image.png&originHeight=138&originWidth=767&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=14728&status=done&style=none&taskId=u633b6dd9-df5a-4f21-a66b-44fe519d577&title=&width=613.6)
其中 P r(ot|xt, Ht) 是衡量当前状态与观察结果吻合程度的观察似然性，而 P r(xt|xt−1) 是衡量当前状态与先前状态差异程度的动态似然性状态。
Observation likelihood.观察似然由两项组成，即均值漂移中心项和边缘项。其公式如下
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34409005/1676117438058-5de9370c-ad17-4fb7-a0cd-c9084451dfae.png#averageHue=%23f6f6f6&clientId=u0138e13d-8cd3-4&from=paste&height=74&id=u3f3eb1a5&name=image.png&originHeight=93&originWidth=862&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=8994&status=done&style=none&taskId=ud0dc01b7-db32-4755-983e-b55a7456fe8&title=&width=689.6)
其中参数 wcen 和 wedge 在实验中分别设置为 3 和 1。
瞳孔均值偏移中心项 Ecen 测量合成虹膜和瞳孔像素 M (rsil) 的均值偏移中心之间的差异，它是使用当前头部方向 Ht 和校准的 3D 眼球模型生成的，与观察到的瞳孔均值偏移中心开场：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34409005/1676117488009-f440b1ca-1270-4dae-96fd-7876133d8764.png#averageHue=%23fbfbfb&clientId=u0138e13d-8cd3-4&from=paste&height=122&id=uf39c6830&name=image.png&originHeight=152&originWidth=692&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=6263&status=done&style=none&taskId=u4cf8c388-165a-4fd2-a523-cda2814a8b6&title=&width=553.6)
直观上，提取的 2D mean-shift 中心很可能接近真实的 2D 瞳孔中心，因此该术语有效地限制了候选瞳孔中心的搜索范围。
边缘项测量渲染图像和观察图像之间边缘图的差异，这也为定位 3D 眼睛瞳孔中心提供了强有力的线索。请注意，边缘图仍然是对真实边缘图的部分观察，尽管我们在提取过程中删除了许多异常值。因此，我们建议使用修剪后的倒角距离度量以获得更好的鲁棒性：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34409005/1676117555180-ae6f2c23-62f9-42b3-896a-b479a574ed26.png#averageHue=%23fafafa&clientId=u0138e13d-8cd3-4&from=paste&height=152&id=u37cd3964&name=image.png&originHeight=190&originWidth=682&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=8482&status=done&style=none&taskId=u4c674979-a9ea-4e41-b433-28239f2a9d5&title=&width=545.6)
其中 I0 d (i) 是观察到的边缘图（图 9）的距离变换，Ir(i) 是渲染的二值边缘图。此项对渲染边缘像素之间的 K 个最小距离求和。 K 由总渲染像素的特定比率 α（在我们的实验中为 0.6）确定：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34409005/1676117577271-2c0e8bac-5639-4fcb-b7a1-f1a8aedb7e66.png#averageHue=%23f9f9f9&clientId=u0138e13d-8cd3-4&from=paste&height=115&id=u58185120&name=image.png&originHeight=144&originWidth=626&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=6872&status=done&style=none&taskId=u3c02001a-6c8a-4532-b313-dc17d5345f2&title=&width=500.8)
请注意，由于时间预算有限，我们不会对渲染的边缘图执行距离变换。此外，如果边缘图不可靠，我们会将边缘项的权重设置为 0，当观察到的边缘像素数低于下限（15 像素）时，这被认为是正确的。
Dynamic likelihood.由于复杂的眼动模式，常用的二阶约束‖xt − 2xt−1 + xt−2‖并不适用。因此，我们提出了一个新的动态似然项，当状态变化很大时，它会自动退化（变得平坦）
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34409005/1676117620214-761dc3ac-2817-42ba-82eb-17d8051231d2.png#averageHue=%23f6f6f6&clientId=u0138e13d-8cd3-4&from=paste&height=90&id=u3cba2b9d&name=image.png&originHeight=113&originWidth=862&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=11143&status=done&style=none&taskId=u89266ece-8477-49c3-aa6b-32dbb7fabcc&title=&width=689.6)
其中 dsphere(xt, xt−1) 定义为两个球坐标之间单位球面上的最小距离，实验中阈值 τ 设置为 0.14 弧度或 8 度，实验中 σ 设置为 0.1。动态似然在状态变化较小时有效地惩罚偏差，例如注视，并在状态变化较大时自动退化，例如眼跳。该术语有效地确保了眼睛注视运动的平滑性，同时允许大的突然变化。
Optimization.因为评估方程式的导数是不平凡的。 12，我们建议通过重要性采样来解决问题。首先，我们使用反向投影的二维均值漂移中心作为均值形成高斯分布，然后用它来对初始候选状态进行采样。标准偏差设置为 0.2。然后，我们使用观察和动态似然（等式 12）重新评估每个候选者的重要性。加权候选实际上是 MAP 框架中真实后验分布的估计。接下来，我们使用此后验分布执行另一轮采样，然后将重新采样的候选状态的加权平均值用作当前 3D 眼睛注视状态。图 10 可视化了重采样后的初始高斯分布和候选权重分布。方差得到有效降低，预期状态明显向地面真值细化，证明了重采样方法的有效性。在我们的实现中，候选者的数量选择为 200。多核 CPU 上的并行实现用于实时性能。
## 6.4 Eye Tracking Failure Detection and Handling
眨眼和闭眼是面部捕捉过程中自然而频繁的动作。当发生眨眼和眼睑遮挡时，二维瞳孔中心检测和边缘图提取的结果变得不稳定。此外，眼睛区域的二维特征检测也可能变得嘈杂和不准确。在这两种情况下，我们的眼睛注视跟踪器的性能都会降低。为了处理这些失败案例，我们引入了两个关于失败检测的新想法，包括闭眼检测器和利用双眼运动依赖性的双眼注视约束。一旦失败检测到，我们将直接使用先前的眼睛注视状态来预测当前帧的输出。
Eye close detector.我们提出了一种用于眨眼事件检测的新型闭眼检测器。它以眼睛图像块作为输入，并返回二进制 true/false 作为输出。当检测到眼睛闭合时，系统将输出当前帧的先前注视状态。与眼睛虹膜和瞳孔分类器一样，我们使用随机森林进行训练和检测。由于当前训练图像数据集包含的闭眼示例非常少，因此我们下载了额外的闭眼人脸示例图像。我们通过对每个样本的选定地标执行随机 2D 相似性变换 10 次来进一步扩充数据库。增强使学习到的树对可能不准确的面部标志更加稳健。我们使用一个有 30 棵树和深度 10 的森林，树训练大约需要 20 分钟。训练和测试数据集中的分类错误率都非常低，表明与眼睛虹膜和瞳孔像素检测相比，眼睛闭合检测要容易得多。我们发现这种策略在实践中效果很好。该组件还用于细化 3D 面部几何形状。一旦检测到闭眼事件并且上眼睑和下眼睑的 2D 界标之间的距离足够小，我们将使用拉普拉斯变形 [Sorkine et al.] 将面部变形的预选上眼睑顶点变形为相应的下眼睑顶点。 2004]，以便正确闭上眼睛。
Double eye gaze constraints.至此，两只眼睛分别被跟踪，可以独立移动。因此，当两个注视跟踪器之一输出无效的眼睛注视状态时，结果可能很糟糕（图 11）。为了确保双眼的运动是有效和自然的，我们提出了一个数据驱动的眼睛注视约束来统计确保双眼依赖。具体来说，我们将双眼的运动表示为一个4值向量（φleft, θleft, φright, θright），并使用k-means算法对注视数据集进行聚类，得到k = 60个数据中心（c1, c2,· · · , CK）。我们认为当前的双眼注视状态 u 是无效的，如果
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34409005/1676118145680-71824eff-d122-4486-86e4-c2a7aae6cfa8.png#averageHue=%23fafafa&clientId=u0138e13d-8cd3-4&from=paste&height=101&id=ubca6aa8d&name=image.png&originHeight=126&originWidth=601&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=4807&status=done&style=none&taskId=u3215ea07-5347-43fe-94a5-c188336a2ba&title=&width=480.8)
其中 γ 由训练数据选择，即 0.2 rad 或 11.45 degree。一旦当前眼睛注视状态被视为异常值，先前的眼睛注视状态将用于预测当前状态。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34409005/1676118130395-0ac481af-b701-45dd-a500-f632f543cadd.png#averageHue=%23b4bba9&clientId=u0138e13d-8cd3-4&from=paste&height=717&id=u2a7d14fa&name=image.png&originHeight=896&originWidth=1009&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=711690&status=done&style=none&taskId=uacf0d7bc-7cca-4626-a3a8-f778b1b8575&title=&width=807.2)
# 7 Evaluations and Results
我们已经在大量视频序列上展示了我们系统的强大功能，包括网络摄像头捕获的实时视频流和从互联网下载的单眼视频序列。此外，我们通过将系统与替代方法进行比较来评估系统的有效性和准确性。我们还评估系统关键组件的重要性。
我们的系统实现了实时性能，并以每秒约 27 帧 (fps) 的帧速率运行。表 1 报告了我们的眼睛注视跟踪器每个关键组件的计算时间。我们所有的实验都在 Intel(R) Xeon(R) CPU 3.3GHz、16GB RAM 和 NVIDIA GTX 780 显卡上进行了测试。我们的结果最好在随附的视频中看到。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34409005/1676118273318-0e73b321-ef2c-463e-b900-2c9f172bfb8d.png#averageHue=%23eeeeee&clientId=u0138e13d-8cd3-4&from=paste&height=318&id=u55eec935&name=image.png&originHeight=398&originWidth=1037&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=43351&status=done&style=none&taskId=u9e4fd693-09dc-4cb5-be93-33b85a56618&title=&width=829.6)
## 7.1 Test on the Real Data
我们已经测试了我们的眼睛注视跟踪器对不同对象的有效性（图 12）。在在线测试中，我们使用分辨率为 800 × 600 的网络摄像头。随附的视频显示，即使在眼睛快速滚动、大头姿势和极端面部表情的情况下，系统也可以非常稳健和准确地跟踪眼睛注视。此外，我们的系统对大的光照变化以及可能的相机模糊也具有鲁棒性（图 13）。除了实时视频流，随附的视频显示我们的系统可以成功地跟踪八个视频剪辑中的眼睛凝视和面部表情，其中六个来自互联网，两个由普通 RGB 网络摄像机记录。
## 7.2 Comparison against Face++
在本节中，我们通过与最先进的 2D 面部特征检测/跟踪系统 Face++ [2015] 进行比较来评估我们系统的有效性和准确性。 Face++是旷视科技开发的商用人脸关键点追踪器。公司免费提供人脸标志检测API，供比对。比较是在八个视频剪辑上进行评估的，包含在各种姿势和照明条件下的不同种族的主题。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34409005/1676118390231-48e702ee-a1e9-4661-b312-47b5747b63eb.png#averageHue=%23937a58&clientId=u0138e13d-8cd3-4&from=paste&height=807&id=u69d85ada&name=image.png&originHeight=1009&originWidth=1071&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=696069&status=done&style=none&taskId=u2c630f37-e851-4e5c-b577-9564bb722d0&title=&width=856.8)
图 14 显示了一些用于比较的示例结果。请注意，Face++ 专注于 2D 面部标志检测/跟踪，而不是 3D 眼睛注视跟踪。出于可视化的目的，我们使用第 4 节中获得的相机参数以及校准过程中的 3D 眼球位置将 Face++ 估计的 2D 瞳孔中心投影到 3D 中。同样，Face++ 的 3D 头部姿势和面部表情变形是基于我们系统重建的
（剩下的部分就是和Face++的各种比较方法和图表）
## 7.4 Applications
Facial and eye gaze retargeting.
Realtime eye gaze capture.
Eye gaze visualization.
























