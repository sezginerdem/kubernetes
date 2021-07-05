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
***
her worker node da 3 temel component bulunur. ilk ve en onemli component containerlarin calismasini saglayacak bir container run time dir. Default olarak bu docker dir ancak k8s docker container run time destegini birakarak bazen containerd ye gecmistir.

* ### kubelet: ornekte ustabasi
her worker node da bulunur. Kubelet api-server araciligiyla etcd yi kontrole eder. Scheduler tarafindan bulundugu node uzerinde calismasi gereken bir pod belirtildi ise kubelet bu podu o sistemde yaratir. Conatinerd ye haber gonderir ve belirlenen ozelliklerde bir container in o sistemde calismasini saglar.

* ### kube-proxy: ornekte lojistik elemani
Olusturulan podlarin tcp, udp ve etcdp trafik akislarini yonetir. ag kurallarini yonetir. kisaca network proxy gorevi gorur.

Bunlarin disinda dns ya da gui hizmeti saglayan cesitli servisler entegre edilebilen servislerdir. fakat bunlar core componentler olarak adlandirilmaz

# Kubectl Kurulumu
***
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

# pod
***
* kubernetes icinde yaratilan en temel obje pod dur.
* kubernetesde olusturabileceginiz en kucuk birimlerdir
* podlar bir yada daha fazla container barindirabilirler ancak cogu duurmda pod tek bir container barindirir
* her bir pod un essiz bir idsi "uid" bulunur
* her pod essiz bir ip adresine sahiptir.
* ayni pod icersindeki containerlar ayni node ustunde calistirilir ve bu containerlar birbirleriyle localhost ustunden haberlesebilirler.
* kubectl ile kubea-api ile haberleserek k8s uzerinde pod yaratma islemi gerceklestirilir. api server bu poda bizim tanimladigimiz bilgileri atar ve bir pod yaratir ve etcd veri tabanina kaydedir. kube-scheduler componenti surekli burayi gozler ve herhangi bir worker node atamasi yapilmamis pod tanimi yapilmamissa o podun calismasi icin uygun bir worker node secer ve bu bilgiyi pod tanimina ekler. Sonrasinda worker node uzerinde calisan kubelet servisi de bu etcd yi surekli gozledigi icin bu pod tanimini gorur ve bu tanimda belirtilen container o worker node uzerinde olusturulur ve boylece pod olusturulma asamalari tamamlanir.

## Pod Yasam Dongusu
#### pending
pod yaratmak icin gonderilen yaml dosyasi kube-api tarafindan alinir ve varsayilan ayarlari da ekleyerek poda bir unique id atar ve tum bunlari etcd ye kaydeder. Bu asamadan sonra pod olusturulmustur ve bu asamadan sonra pending asamasina gecer. Eger bir podun statusu pending durumunda ise bunun anlami sudur: birisi bir pod olusturdu podla ilgili tanimlar yapildi ve veri tabanina kaydedildi ama pod herhangi bir node uzerinde olusturulmadi. 

#### creating
kube-scheduler api-server araciligiyla surekli etcd yi gozler. eger burada yeni yaratilmis ve herhangi bir node atamasi yapilmamis bir pod gorurse calismaya baslar. Algoritmasi ve secme kriterlerine gore podun calismasinin en uygun olacagi node u secer ve veri tabanindaki pod objesine node bilgisini ekler. Bu noktadan itibaren pod yasam dongusunde creating asamasina gecer. eger bir pod creating asamasina gecemiyorsa bu kube-scheduler in uygun bir node bulamadigi anlamina gelir. bunun pek cok nedeni olabilir: cluster da bu podun gereksinimlerini karsilayabilecek node bulunmuyor olabilir. Nodelar uzerinde cpu ve memory gibi kaynaklar tukenmis olabilir. 

#### ImagePullBackOff
Nodelar uzerinde kubelet adli bir servis calisir. bu servis ayni kube-scheduler gibi surekli etcd veri tabanini gozler ve bulundugu node a atanmis nodelari tespit eder ve hemen islemlere baslar. Ilk olarak pod taniminda olusturulacak containerlara bakar ve bu containerlarda olusturulacak image lari sisteme indirmeye baslar. Eger bir sekilde image indirilemezse pod erimagepull ve ardindan da imagepullbackoff asamasina gecer. Eger podun statusunda imagepullbackoff gorurseniz bu nodeun image i repository den cekemedigi ve bunu tekrar tekrar devam ettigi anlamina gelir. Bunun birkac nedeni olabilir: en sik karsilasilan neden pod tanimlanmasinda image isminin yanlis yazilmasidir. Image ismi yanlis yazildigi icin image cekilemeyecektir ya da image i cekmek icin sisteme kaydolmak gerektigi ve bu authentication islemlerinde hata yapilmasidir. Bu nedenle image cekilemeyecektir. Bu gibi bir hata olmamasi durumda ise

#### Runnig
Islemlerin sorunsuz bir sekilde ilerlemesi halinde kubelet node da bulunan container engine ile haberlesir ve ilgili containerlarin olusturulmasini saglar ve containerler running durumuna gecer. Bu noktadan itibaren artik pod olusturulmus olur.

* Containerlarla ilgili temel kural: container icindeki uygulama calistigi surece container calisir. Uygulama calismayi birakirsa container durdurulur. Uygulamanin da durdurulmasi 3 sekilde olur: 1) isi bittigi icin hata vermeden otomatik olarak kapanir. 2) Kullanicinin istegi uzerine komut gonderilir ve hata vermeden kapanir. 3) hata verir, sikinti cikar, hata kodu olusturarak kapanir. Ornegin nginx image uzerinden bir container yarattigimizi dusunelim. Bu uygulama bir deamon bir servis yani siz onu durdurana ya da hata cikip cokene kadar calismaya devam eder. O calistigi surece de container alismaya devam eder. Fakat diyelim ki siz bir image yarattiniz ve bu image in varsayiln uygulamasi da bir script ya da basit bir komut. container baslayinca bu script calisiyor isini yapiyor ve isi bitince kapaniyor yani container da kapaniyor yani illa ki hata vermeden kapanmasina gerek yok.

Bu gibi sorunlar nedeni ile containerlar icin restart policy tanimi yapilir. Restart policy 3 adet deger alabilir:
* Always: Default degerdir. Pod un icerindeki container her durumda durdurulursa durdurulsun her sartta yeniden baslat anlamina gelir. 
* On-failure: Sadece hata alip kapanirsa yeniden baslatilir.
* Never: Pod hicbir zaman yeniden baslatilmaz.

#### Succeed
Podun altindaki containerlar calismaya devam ettikce status running olarak devam eder. Eger containerlarin hepsi hata vermeden dogal olarak kapanirsa ve restart policy never veya onfailure olarak set edildi ise podun statusu succeed olarak statusune gecer ve pod yasam dongusunu tamamlar. Yani bir diger degisle completed, basarili olarak pod un yasam dongusu tamamlanir. 

#### Failed
Never ya da Onfailure olarak policy tanimlandi ve containerlardan biri hata verip kapandi ise bu sefer pod un statusu failed olarak isaretlenir ve yasam dongusunu boyle tamamlar. 

#### CrashLoopBackOff
fakat restart policy always olarak set edildi ise pod hicbir zaman succeed ya da failed durumuna gecmez. Bunun yerien podun icerisindeki container yeniden baslatilir ve running state de devam eder. Fakat kubernetes bu yeniden baslatma islemini belirli bir siklikla yapiyorsa bazi seylerin ters gittigine kanaat getirilir ve pod u CrashLoopBackOff adini verdigimiz bir state e sokar. Bunun anlami sen bir pod olusturdun ama bu pod un icerisindeki container ikide bir kapaniyor ama gene kapaniyor buna bir bak. Bu pod da sunlar olur: k8s container in icerisinde birseylerin ters gittigini anlar ve pod un statusunu crashloopbackoff a cevirir. Surekli restart etme yerine restart eder 10 sn bekler eger 10 sn icinde yeniden cokerse 20 sn bekler yeniden cokerse 40 sn bekler sonra 80 sn bekler ve bu durum 5dk lik araliga cikana kadar boyle devam eder ve ondan sonra her 5dk da bir tekrar eder. Bu arada container cokmeyi birakir ve 10dk sure ile sorunsuz bir sekilde calismaya devam ederse. Kubelet container i crashloopbackoff dan cikarir ve running e dondurur. eger bu olmaz ve siz mudahale etmezseniz bu durum sonsuza kadar boyle devam eder. Ozetle crashloopbackoff mudahaleyi gerektirir ve bakilmasi gerekmektedir. 