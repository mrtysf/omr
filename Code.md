# omr
from imutils.perspective import four_point_transform
from imutils import contours
import numpy as np
import imutils
import cv2

Cevap_Anahtari = {0: 1, 1: 4, 2: 0, 3: 3, 4: 1, 5: 2, 6: 3, 7: 1, 8: 3, 9: 1,
                  10: 3, 11: 0, 12: 1, 13: 3, 14: 2, 15: 4, 16: 4, 17: 1, 18: 2, 19:2,
                  20: 0, 21: 1, 22: 2, 23: 3, 24: 4, 25: 3, 26: 2, 27: 2, 28: 2, 29: 1,
                  30: 1, 31: 2, 32: 3, 33: 4, 34: 0, 35: 0, 36: 1, 37: 2, 38: 4, 39: 0}

foto = cv2.imread('3.jpg')
gri = cv2.cvtColor(foto,cv2.COLOR_BGR2GRAY)
bulanik = cv2.GaussianBlur(gri, (5, 5), -1)
kenar = cv2.Canny(bulanik, 30,10)

#kontur bulma, External=sadece dış hatları alıyor
#Approx_simple=yatay dikey çizgilerin köşeleri alır
kontur = cv2.findContours(kenar.copy(), cv2.RETR_EXTERNAL,
	cv2.CHAIN_APPROX_SIMPLE)
kontur = kontur[0] if imutils.is_cv2() else kontur[1]
optikkontur = None
#En az 1 tane kontur bulduğumuzdan emin oluyoruz. len(sayısı)
if len(kontur) > 0:
        #Konturları büyükten küçüğe sıralıyoruz. Büyük olanı alıyor ilk.
	kontur = sorted(kontur, key=cv2.contourArea, reverse=True)
         # for methodunda da bu sıralamada büyüğü
	for c in kontur:
		cevreuzunluk = cv2.arcLength(c, True)
		yaklasik = cv2.approxPolyDP(c, 0.02 * cevreuzunluk, True)
 
		if len(yaklasik) == 4:
			optikkontur = yaklasik
			break
#algılanan dikdörtgen alanın perspektif dönüşümü
optik = four_point_transform(foto, optikkontur.reshape(4, 2))
grioptik = four_point_transform(gri, optikkontur.reshape(4, 2))
thresh = cv2.threshold(grioptik, 0, 255,
	cv2.THRESH_BINARY_INV | cv2.THRESH_OTSU)[1]

kontur = cv2.findContours(thresh.copy(), cv2.RETR_EXTERNAL,
	cv2.CHAIN_APPROX_SIMPLE)
kontur = kontur[0] if imutils.is_cv2() else kontur[1]
sorukontur = []

for c in kontur:
    (x, y, w, h) = cv2.boundingRect(c)
    kutu = w / h

    if w >=5 and h >=5 and kutu >=0.4 and kutu <=0.6:
      sorukontur.append(c)
#sıralama yapıyoruz üstten altta doğru
sorukontur = contours.sort_contours(sorukontur,
	method="top-to-bottom")[0]
dogru = 0
#Soldan sağa sıralama ve konturları ayırma işlemi
#her sorunun 5 olası cevabı 

for (q, i) in enumerate(np.arange(0, len(sorukontur), 5)):
	#soru için konturları sıralama işlemi soldan sağa devam ediyor
        #5 olasılığa bakılınca alt soru geçip aynı işlem ediyor 
	kontur = contours.sort_contours(sorukontur[i:i + 5])[0]
	isaretli = None
        #sıralı konturları için tekrar döngü yapıyoruz
	for (j, c) in enumerate(kontur):
		#maske oluşumu yapıyoruz her bir şık için
                #siyah ekranda uyguluyoruz ve çizdiyoruz
		maske = np.zeros(thresh.shape, dtype="uint8")
		cv2.drawContours(maske, [c], -1, 255, -1)
 
		#thresh görüntümüzü kullanarak maskelediğimiz
		#Tüm şıkları için 0dan fazla olan pikselleri saydırıyoruz
		maske = cv2.bitwise_and(thresh, thresh, maske=maske)
		toplam = cv2.countNonZero(maske)
 
		#0 dan farklı piksel varsa incele işaretli cevabı kullan
		if isaretli is None or toplam > isaretli[0]:
			isaretli = (toplam, j)






#bulunan konturun çevre çizimi
cv2.drawContours(foto, [optikkontur], -1, (0, 255, 0), 2)
#cv2.imshow('işaretlenmiş', maske)
cv2.imshow('cevapkağıdı',optik)
cv2.imshow('gricevapkağıdı',grioptik)
cv2.imshow('gorunum',foto)
cv2.imshow('kenar',kenar)
cv2.imshow('thresh',thresh)
cv2.imshow('bulanık',bulanik)
cv2.waitKey(0)
cv2.destroyAllWindows()

