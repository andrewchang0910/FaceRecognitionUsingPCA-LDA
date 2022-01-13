# Python PCA LDA 臉部辨識 (SK-learn)
#### [FaceRecognitionUsingPCA&LDA Github Link](https://github.com/Aaron-Ace/FaceRecognitionUsingPCA-LDA)

### 實作作方法與步驟：

1. 依據資料的影像格式，讀取pgm影像檔。
2. 每張人臉影像均為92x112=10304的灰階影像，將其轉為10304x1的向量，即成為一個樣本。
3. 資料共含有400張影像（40人，每人10張），訓練時只用200張（每人隨機取5張）。
4. 利用PCA計算此200張影像的轉換矩陣，設法將維度從10304降至10, 20, 30, 40, 50維
5. 以較低維度的樣本訓練出SVM分類器來進行辨識。
6. 比較不同維度的辨識率，並統計出混淆矩陣
7. 以降維後的樣本，利用FLD(LDA)找出另一轉換矩陣，轉為有較佳的類別分離度之新樣本。
8. 以SVM辨識器再次評量辨識率以及統計混淆矩陣。

### 資料說明：

* att_faces資料集，由劍橋大學AT&T實驗室創建,包含40人共400張面部圖像,部分志願者的圖像包括了姿態,表情和面部飾物的變化。大小是92×112(Face recognition database, a total of 40 individuals, each person 10 images, size is 92×112)。

### 程式架構圖：

![](https://i.imgur.com/njq6qLq.jpg =x50%)

### PCA 模型：
```
    def PCA_model(dimension, xTrain, xTest, yTrain):
        pca = PCA(n_components=dimension)
        pca.fit(xTrain, yTrain)
        xTrain = pca.transform(xTrain)
        xTest = pca.transform(xTest)
        return xTrain, xTest

```
### LDA 模型：
```
    def LDA_model(xTrain, xTest, yTrain):
        lda = LDA()
        xTrain = lda.fit_transform(xTrain, yTrain)
        xTest = lda.transform(xTest)
        return xTrain, xTest

```

### SVM 分類器：

```
    def recognizer(dimension, xTrain, xTest, yTrain, yTest):
        svm = SVC(kernel='linear')  # 支援向量機方法
        svm.fit(np.array(xTrain), np.float32(yTrain))
        yPredict = svm.predict(np.float32(xTest))
        # print('really: {}'.format(np.array(yTest)))
        # print('predict: {}'.format(yPredict))
        print('維度%d: SVM向量機識別率: %.2f%%' % (dimension, (yPredict == np.array(yTest)).mean() * 100))
        return yTest.tolist(), yPredict
```
    
### 混淆矩陣：

```
    def plot_confusion_matrix(name, dimension, confusion_mat):
        plt.imshow(confusion_mat, interpolation='nearest', cmap=plt.cm.gray)
        plt.title('{} Dimension {} Confusion matrix'.format(name, dimension))
        plt.colorbar()
        tick_marks = np.arange(4)
        plt.xticks(tick_marks, tick_marks)
        plt.yticks(tick_marks, tick_marks)
        plt.ylabel('True label')
        plt.xlabel('Predicted label')
        plt.show()
```

### 辨識結果：
* 分為兩列，第一列為PCA結果，第二列為LDA結果

![](https://i.imgur.com/kIBLnjz.jpg =100%x)

根據上圖可以發現經過PCA降維之後的樣本與再經過LDA降維的樣本相比
後者的辨識度幾乎都有提升，除了降成10維的樣本沒有符合這個規律

計算多次的之後結果依然，我認為是因為維度太低，無法有足夠的特徵去有效辨認樣本

這個論點可以根據維度慢慢提升，後者的辨識率都有慢慢提升而得到驗證。

### 混淆矩陣結果：

![](https://i.imgur.com/gtla1nP.png =50%x)![](https://i.imgur.com/XloEabw.png =50%x)
* 根據上面兩張10維的混淆矩陣可以發現，LDA的版本雖然有效的修正了一些不良的特徵，但是卻也新增了一些不完整的特徵，造成結果不能優於PCA的版本。
![](https://i.imgur.com/UISP6pD.png =50%x)![](https://i.imgur.com/MndAyMC.png =50%x)
* 根據維度的慢慢提升，LDA版本的表現慢慢高於PCA的表現。
![](https://i.imgur.com/bC3yjHX.png =50%x)![](https://i.imgur.com/qeFsd9o.png =50%x)
* 錯誤的比數越來越低
![](https://i.imgur.com/BlhXuwS.png =50%x)![](https://i.imgur.com/wJc71Yk.png =50%x)
* 維度升高特徵比較充足，整體表現都有提升
![](https://i.imgur.com/wyCam5v.png =50%x)![](https://i.imgur.com/JnClWnZ.png =50%x)

### 結果探討與心得

這次的作業內容還蠻特別的，一般人臉辨識的像素特徵都會以較多的數量以當前最完整的方式去做辨認偵測，這樣數學的模式將其降維之後再作辨認，沒想到效果也很好，辨識率竟然也可以達到99%讓我有點驚訝，但是我不太知道如果資料量大了之後會不會造成辨識率降低，因為特徵不夠多。

整體而言，這次使用Sk-learn套件讓我的作業簡單的許多，另外PGM的影像檔是以往沒有使用過的圖像類型，是個特別的體驗。












