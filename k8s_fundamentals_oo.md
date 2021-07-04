# Kubernetes Yonetim Komponentleri

## CONTROL PLANE-MASTER NODE

* ### Kube-apiserver: Resepsiyon ornegi
kubernetes ile alaki um komponentlerin ve dis dunyadan kubernetes platformu ile iletisim kuran tum servislerin ortak giris noktasidir. Api server tum komponent ve node bilesenlerinin direk iletisim kurabildigi tek komponenttir. Tum iletisim api server uzerinden gerceklestirilir. Kube-apiserver k8s de kaynak olusturma islemlerinde api dogrulamasindan sorumludur. Kullanicilar kube ctl komut satiri istemcisi veya rest-api cagrilari araciligiyla api server ile iletisim kurabilirler. Kisacasi disaridan istek gerceklestirmek icin apiserver a ulasir authantication ve authorization islemlerini gercektlestirir ve kubernetese ulasirsiniz. Diger tum komponenetlerde isteklerini api server uzerinden gerceklestirir.

* ### Etcd: 
tum cluster verisi metadata bilgileri ve k8s de olusturulan tum objelerin bilgilerinin tutuldugu anahtar-deger “key-value” veri deposudur. Clusterin mevcut durumu ile ilgili bilgiyi uzerinden barindirir. Kisacasi k8s in calismasi icin ihtiyac duyulan tum bilgiler etcd uzerinde tutulur. Apiserver haric baska hicbir komponent etcd ile dogrudan haberlesemezler. Etcd ile iletisim kurmak istediklerinde bunu api-vserver uzerinden gerceklestirirler. 

* ### Kube-scheduler: ornekte uretim planlama. 
Yaratilmak istenen podlarin gereksinimlerini kontrol ederek o podun en uygun calisacagi worker node un hangisi oldugunu belirler ve o podun burada olusturulmasini saglar. Podun ozel gereksinimerini isteklerini, cpu gibi cesitli parametreleri elinde bulundurarak degerlendirir ve pod icin en uygun node un hangisi olduguna karar verir. 

* ### Kube-controller-manager: ornekte vardiya amiri
Tek bir binary olarak bulunsa da icinden birden fazla kontroller bulundurur. K8s istenilen durumla mevcut durum arasindaki karsilastirmada uygulama yonetimi saglar. Kube-controller altindaki controller managerlar k8s den istenilen durumla mevcut durum arasinda fark olup olmadigi gozlerler. Kube-api araciligiyla etcd de saklanan cluster durumunu inceler ve eger mevcut durum ile istenilen durum arasinda fark varsa iste bu farki olusturan kaynaklari gerektigi gibi olustururak, guncelleyerek veya silerek bu durumu esitler. Ornegin siz k8s e uygulamanizin 3 pod olarak calismanizi bildirdiniz. K8s de bunu gerceklestirdi ve uygulamanizin kostugu 3 pod calismaya basladi ama sonra bir tanesi silindi iste controller-manager bu podun yeniden olusturulmasini saglar. 

Control-plane yani yonetim kismi master node uzerinde calisir. Bu componentlerin hepsi tek bir linux isletim sistemine kurulabilecegi gibi birden fazla sisteme de kurulabilir. etcd tamamen bu yapidan ayri bir sekilde konuslandirilarak daha yuksek bir erisim saglanabilir iken ama genellikle etcd de api-server in kostugu yerde durur. ornegin 3 sunucu uzerinde 3 api-server kurulup diger componentler bu sistemlere dagitilabilir. Bu uzerinde control plane componentlerinin kostugu sistemlere cluster altinda master node deriz. Master nodelar sadece bu sistemleri barindirmak icin bulunur ve production ortaminda herhangi bir is yukumuzu bu sistemler uzerinde calistirmayiz. Bunlarda sadece cluster in yonetim altyapisini kostururuz. is yuklerimiz ise worker nodelar uzerinde calistirilir. Bunlar uzerinde bir container run time barindiran ve clustera dahil edilmis sistemledir. 

## WORKER NODE
her worker node da 3 temel component bulunur. ilk ve en onemli component containerlarin calismasini saglayacak bir container run time dir. Default olarak bu docker dir ancak k8s docker container run time destegini birakarak bazen containerd ye gecmistir.

* ### kubelet: ornekte ustabasi
her worker node da bulunur. Kubelet api-server araciligiyla etcd yi kontrole eder. Scheduler tarafindan bulundugu node uzerinde calismasi gereken bir pod belirtildi ise kubelet bu podu o sistemde yaratir. Conatinerd ye haber gonderir ve belirlenen ozelliklerde bir container in o sistemde calismasini saglar.

* ### kube-proxy: ornekte lojistik elemani
Olusturulan podlarin tcp, udp ve etcdp trafik akislarini yonetir. ag kurallarini yonetir. kisaca network proxy gorevi gorur.

Bunlarin disinda dns ya da gui hizmeti saglayan cesitli servisler entegre edilebilen servislerdir. fakat bunlar core componentler olarak adlandirilmaz

# Kubectl Kurulumu
Kubernetes e komut vermek icin bubu api-server uzerinden yapiyoruz. 
Bunun 3 yontemi var. 

* #### Rest cagrilari
Uygulamalarin ya da scriptlerin icinde kullanilir
* #### Gui istemcileri
Grafiksel arayuz ile iletisim kurmak. Resmi gui araci dashboard, oktant, land en bilinen araclardir. Gui istemciler en temel araclar degildir. 
* #### kubectl
Shell uzerinden apiserver a komutlar doderdigimiz k8s in resmi cli aracidir. k8s cluster olusturulmadan once ilk olarak kubectl yuklenmesi gerekir.

## kubectl config dosyasi
* kubectl araci baglanacagi kubernetes cluster bilgilerine config dosyalari araciligiyla erisir
* config dosyasinin icerisinde kubernetes cluster baglanti bilgilerini ve oraya baglanirken kullanmak istedigimiz kullanicilari belirtiriz
* Daha sonra bu baglanti bilgileri ve kullanicilari ve ek olarak namespace bilgileirni de olusturarak contextler yaratiriz
* kubectl varsayilan olarak $HOME/.kube/ altindaki config isimli dosyaya bakar
* kubectl varsayilan olarak $HOME/.kube/ altindaki config dosyasina bakar ama bunu KUBECONFIG environment variable degerini degistirerek guncelleyerebilirsiniz
