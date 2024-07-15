## PROJEYE HAZIRLIK ADIMLARI

STEP-1:

terraform dosyalarını çalıştır.
4 makine oluşur. 
postgersql, nodejs, react, control-nodejs

STEP-2:

Ccontrol-node ' a SSH ile bağlan ve,
ansible all -m ping #makinelere ping atabiliyorsak çalışmaya başlayabiliriz.
ansible all -m ping -o # tek satır halinde makinelerden pong döner.

STEP-3:

eğer ssh ile bağlandığın control node' a "ansible.cfg" , "inventory_aws_ec2.yaml dosyası,
bir de .pem uzantısı ile beraber pem-key dosyan geldiyse tamamdır.Not: dikkat key ".pem" 
uzantısı ile geldiğine emin ol.bazen KEY gelmeyebiliyor.

STEP-4:

VSCode Ansible extension yükleyelim.

STEP-5:

## PROJENİN RESİMDEN AÇIKLAMASI:

bu uygulama Nodejs uygulaması. PERN diye de kısaltması geçer .yani
postgersql
expresjs
react
nodejs

MERN kısaltması ile de geçer . postgersql yerine, MONGODB kullanıyorsak MERN denir.

PROJE nin resmine bakarsak :
CONTROL NODE:

bir tane control node var bunu zaten terraform ile kurduk. terraform ile kurarken EC2 FULL access yetkisini vermiştik.Dynemic inventory kullanıyorsak
1- ec2 full access
2- Boto3 
yüklü olmalı ki EC2 full access yetkisine binaen AWS ye gidip instance lerin bilgilerini alabiliyor.
control node 'da 
1-config file
2-inventory
3-pem file
4-Playbook da biz yazacağız.

3 makine daha kurulmuştu terraform ile.

1-postgersql : todo uygulamasının datalarını tutuyor.
2-nodejs: backend kısmı : datayı oluşturup postgresql 'e aktarıyor. json formatında bir API
3-react: frontend. todo arayüzünü gösteriyor.
#NOT : Biz bu 3 makineyi tek makine de de toplayabilirdik docker compose ile de yazabilirdik. ama burada ansible ile configüre edilmeyi örnekliyoruz.

3 makinenin herbiri bir container ama herbiriayrı bir EC2 ya kuruluyor.ansible ile configüreyi anlayabilmemiz için.

 code dosyalarındaki:

client=react
database=postgresql
server=nodejs 

NOT: control node t3 a midium diğer 3 makine t2 micro. DİKKAT!!!! Bazen react makine image ları
oluştururken sıkıntı çıkarabiliyor ozaman gidip react makine type ini yükseltmek gerebilir.
ozaman makineyi stop edip makinenin type ni değiştirebiliriz.

STEP-6:
sorcecode ları yükle Dockerfile larını yaz ve image larını oluştur.
source code lar developer lar tarafından yazılmış . biz bunların dosyalarını control-node yükleyip Dockerfile 'larını yazacağız.

STEP-7:

İMAGE ler hazır olduktan sonra
Playbook ları yazacağız. her bir makine için ayrı playbookları yazacağız.

STEP-8:

## PROJE NIN TASK INI README DEN incelersek.:

1-port lar zaten hazır. Terraformla kurarken zaten açtık.
Nodejs:5000
postgersql:5432
react:3000
***bunlar proje Readme de var zaten.

2-sorcecode lar ilgili makinelere gönderilecek.
3-bütün node lar control-node dan yönetilecek.
3-Dynemic inventory kullanıyoruz.
4-Ansible config file hazır
5-Docker, ansible kullanılarak tüm makinelere kurulmalı

postgresql:

1- sorcecode ları çekip Dockerfile ları yazıp image ları oluştıracağız.önce postgersqliçin yazacağız. Tabi init.sql bize verilmişti zaten .o da dahil edilmeli.
2- postgersql için gerekli env leri ansible vault ile oluşturup ,postgersql container ı ayağa kaldıracağız.
3- sec grp lar zaten terraformla ayarlanmış oldu.
4- container kapanınca herhangi bir nedenle bilgiler gitmemesi için volume bağlayaağız.

Nodejs worker node:backend

1-dataları oluşturup postgersql'e aktaracak. bu yüzden postgersql 'in private ip sini Nodejs 'in env dosyasına girmem gerekir.
aynı şekilde react da nodejs ile konuşabilmesi için : nodejs public ip sini ,react 'in env dosyasına  girmem lazım.
2-image oluştur
3-port 5000 zaten açık (terraform)
4-sec-grp(terraform) ayarlanmış

React worker node:frontend

1- env file a nodejs public ip sini yazacağız
2- source codeları gönder . Dockerfile yaz. image hazırla.
3- port :3000 - 80 zaten(terraform)
4- sec-grp zaten(terraform)
uygulamayı 3000 portundan görürüz.


control node için 

-installing Ansible
-ansible.cfg
-inventory_aws_ec2.yaml(Dynemic-inventory) .Dynemic-inventory   dosyası mutlaka şu şekilde biter  "_aws_ec2.yaml"

bu 3 gerekli olmazsa olmazı terraform ile kurduk.
şimdi;

STEP-9: 

3 tane Dockerfile 
3 tane Playbook yazacağız.

-------------------------------------
PLAYBOOK LAR :
---------------------------------------


postgresql için:

Playbook'da şunlar var:

update
install Docker
initsql copy ile gönder
Dockerfile gönder
build image
run container
bazı extra işlemler(eski container ve image ları silme gibi)

YANİ: 

-installing docker
-initsql yani source code u önce Github dan controlnode 'a alacağız., kopyalayacağız. Sonra buradan ilgili server a 
Playbook yazarak göndereceğiz.Copy module kullanacağız.
-önceden yazdığım Dockerfile playbook da copy modulünü kullarak  ilgili servera gönder . 
-sonra playbook da build image diyoruz.
-sonra yine playbook da run container diyeceğiz 

****************ÖZET-ÖZET-ÖZET***************
ÖZETLE TÜM YAPACAĞIMIZ İŞ ŞU:
1- controlnode da PROJE folder oluştur. 
onun altında 3 adet folder oluştur.
postgresql
nodejs
react
2- bu folder lara ilgili sorcecode ları koyacağız.Dockerfile ı orada yazacağız.ilgili server a gönder.
3- ilgili serverda önce docker kuracağız.
4- ilgili dosyaları control node dan ilgili server a gönder
5- build image 
6- run container
 
nodejs:
playbook için:

update
install Docker
server files copy  gönder
Dockerfile gönder
build image
run container
bazı extra işlemler

react:
playbook için:

update
install Docker
client files copy  gönder
Dockerfile gönder
build image
run container
bazı extra işlemler


extra işlemler:  env dosyasındaki " ip " lerin değiştirilmesi gibi
---------------

********************inventory dosyası hakkında *****************************
----------------------------------------------------------------------------
filter lar ve key_group lar kullandık . doğru hostlara ulaşmak için doğru filter yapmalıyım. 
gruplamaları yaparken kullandığım tag leri vermeyi terraform yazarken düşünmeliyiz. 

PROJE ADIMLARI


şimdi projeye başlayalım:
*************************************
STEP-1 :

control-node a bağlan.ve ana-dizinde
-mkdir ansible
-cd ansible
-mkdir ansible-project playbooks && cd ansible-project && mkdir postgres nodejs react
-ls -R # dosya yapısının ayrıntısını verir.
ya da tree yi inidrip görebilirsin:
-sudo dnf install tree -y

STEP-2:

şimdi souce code ları bu gerekli folder lara kopyalayacağız.
Dockerfile ları da burada yazacağız.
daha sonra playbookları yazarken bu dosyaları buradan al gerekli serverların içerisine ansible ile göndereceğiz.
playbokkları da playbooks  dosyası altında yazacağız.
-daha rahat çalışmak için oluşturduğum ansible dosyasına geçebilirim solda dosya yapısı karışık olmasın istersem.
-şimdi 
postgres :
source-code
Dockerfile
student_files altındaki todo-app-pern dosyasının içindeki database altındaki initsql i ,oluşturduğum 
ansible altındaki postsgres dosyasına sürükle at. şimdi postgres altına (newFile) Dockerfile ını yazacağız.

STEP-3:

postgres Dockerfile 'ı:
****************************
FROM postgres
COPY ./init.sql /docker-entrypoint-initdb.d/  # dockerhub posgres image açıklamasından buldum(initialization scripts)
EXPOSE 5432 # yazılmasa da olur bu sadece bilgilendirmedir.

 STEP-4:

*playbook for postgres:

DİKKAT EDİLECEK HUSUSLAR:
 
01-hosts:  bu kısma hangi makineyi configüre edeceksek onu yazıyoruz.

bütün makinelerimin adlarını da şu komutla görebilirim.
-ansible-inventory --graph
_ansible_postgresql **bu şkilde makinemin adını gördüm ve tasklarımı yazmaya başlıyorum.

src-code ve Dockerfile hazır şimdi,
playbook dosyasına postgres için playbook yazacağız.

02-bunun için playbook dosyasına bir docker_postgres.yml file oluştur.

03-playbook da task olarak:
1-update (yum paketine göre. çünkü bu rethat makine centos ile aynı "yum" paketi kullanıyor.)
2-Docker install edeceğiz çünkü imagebuild edecek.
bunun için docker install for centos(rethat makina aynı bununla yani yum paket maneger kullanır.) Documentation a git bak.
orada önce uninstall old version diyor .sonrada installation için command ları veriyor.
docker repo yu ekliyoruz
docker engine yüklüyoruz 
ec2-user ı docker group a ekliyoruz .
dockerı service i start ediyoruz. ediyoruz.

*şimdi docker kuruldu

3-artık copy modülü ile  init sql ve docker file ı gönderme taskını yazacağız playbookta:
çünkü image ı postgre server'ında build edeceğiz.file ları oraya göndermem gerek.
artık build oluşturacağım .ama önce var olan image ları silme task ı yazmam gerek.
çünkü developer lar image değiştirince bu image silme adımı olmazsa ansible ın"idompotency" özelliği sebebiyle yinelenen çağrılarda değişiklik yapmaz.
ansible "zaten böyle bir image var ve böyle bir container var" diyerek skipp eder bu taskı atlar yapmaz. bu nedenle
önce var olan image ları silme sonra build etme taskını yazarız. "when "condition  da kullanılabilir.
veya her image a dynamic olarak farklı isim vererek de yapabiliriz 
ama bir zaman sonra mesela bunu 10 defa çalıştırınca pekçok işe yaramayan 
eski image lar makineye yük olacak ve makineyi şişirecek ve makine patlayacak.

bu sebeplerle biz burada 
önce image silme ve sonra image build etme taskları yazacağım.
ama ilk çalıştırdığımda containerda olacağı için en önce container silme task ı yazmam gerek.
Yani; tasklar şu sırayla:
4-cont sil
5-image sil
6-image build 
7-container çalıştır. bu taskı yazarken env ler gerekiyor. bunları dockerhub postgersql doc ten bulurum. 
orada  POSTGRES_PASSWORD env sadece required diğerleri optional.Bu env 'ler için server dosyasındaki nodejs için olan
 env ye bakmam gerek.
çünkü benim nodejs ile postgresql birbirine bağlanacak.dolayısıyla ben nodejs altındaki env lere bakmam gerekir.
oradan
da sadece  DB_PASSWORD ü alıyorum tek gerekli env olduğundan .diğer env ler ben nodejs image oluştururken Dockerfile ın 
olduğu yerde env file ı da olacak ve ben image ı oluştururken o env leri image ın içine gömeceğim bu yüzden env lere sahip olan
nodejs ,direk gidip postgresql ile bağlantıyı kurabilecek.

8-sıra volume bağlama taskını yazmada
postgersql image doc de volume bağlama yeri :
/var/lib/postgresql/data 
olduğunu buluyorum.
ben de volume için bu ad da bir dosya oluşturup bağlıyorum"/db-data"# bu ismi biz verdik.readme de proje taskında belirlenmiş.


volumes:
  - /db-data:/var/lib/postgresql/data

 STEP-5:

*******ANSİBLE VAULT******

password açıkta kullanılması uygun olmaz .
ansible vault kullanacağız.

cd playbooks deyip playbooks dosyasında iken

-ansible-vault create secret.yml  # komutunu çalıştıracağız.
password verip confirm et.
vim açılır .
password ümü buraya giriyorum.
password: Pp123456789 
save edip çıkıyorum.bunu yapınca secret.yml oluştu.
tekrar görmek istersem şu komut gir.
-ansible-vault view secret.yml
bana sorduğu şifremi girersem password 'ümü görebilirim açık şekliyle.

STEP-6
 
artık postgre nin playbook da task da POSTGRES_PASSWORD ün yerine direk yazmıyorum.
POSTGRES_PASSWORD: "{{password}} şeklinde giriyorum.

STEP-7:

secret.yml ı da playbook da göstermem gerek.
become: true altına gelip
vars_files:
   -secret.yml

şeklinde girmem gerek.

STEP-8:

PLAYBOOK ÇALIŞTIR:
ansible-playbook --ask-vault-pass docker_postgre.yml

******DİKKAT!!!!!!!????????????************

ansible playbook secret.yml yani ansible-vault kullandığım için şu komutla çalışmaz.
-ansible-playbook docker_postgre.yml # bu komutla çalışmaz çünkü vault kullandım.
komutta vault password girmem gerek. bunun için --ask-vault-pass parametresini girmem gerek.
-ansible-playbook --ask-vault-pass docker_postgre.yml # Bu komutla vault password ümü ister.
password ümü girince playbook çalışır.

***NOT:
makinaya gidip bağlanmadan biz ansile/playbooks dosyasında iken image ım oluşmuşmu configüre ettiğim makinede nasıl görebilirim?

ansible _ansible_postgresql -m shell -a "sudo docker images ; sudo docker ps

video 2:37

STEP-9:

aynı işlemleri nodejs için yapacağız.
nodejs:

postgres için yaptıklarımızın aynışarını burada tekrar yapacağız.

1-server dosyası altındaki dosyaları nodejs dosyasına atıyoruz.
2-env dosyasında DB_HOST kısmına ,postgersql makinenin privat ip sini aws console dan kopyala yapıştır.
3-Dockerfile yazıyoruz..package.json ,nodejs için gerekli paketler , modüller, dependensy ler kütüphaneler.container ın çalışma dizinine(./)
kopyalıyoruz.nodejs in tercih sebeplerinden biri budur. pek çok modul  var kütüphanesi çok geniş.
npm nodejs in paket yöneticisi.Bunların developerlarla toplantılarla öğrenilmesi gerekir.

4-şimdi nodejs için playbook yazacağız:

01-playbooks altına geç ve docker_nodejs.yml yi oluştur.

02-bir önceki postgre ile aynı sadece şunları değiştiriyoruz:
***********değişiklikler***********************************
--------------------------------------------------------------
hosts adı ,configüre edeceğin makinenin adı
src code lar  ve Dockerfile
image ve container adı
port
5***NOT : tabi env de mutlaka postgresql makinenin private ip sini girmeyi unutma.

artık bu playbook da hazır çalıştıralım.
-ansible-playbook docker_nodejs.yml
çalıştıkdan sonra container ım çalışmışmı kontrol edelim.
-ansible _ansible_nodejs -m shell "sudo docker ps"
çalışıyorsa gidip browserda nodejs-public-ip:5000 port undan görürüz .
ekranda sadece 
Canot GET / yazısı geliyor sadece .çünkü daha frontend imiz hazır değil. arayüz göremiyoruz.
nodejs-public-ip:5000 /todos   yazarsam da sadece bu gelir:[]. çünkü data yok.
ama data girebilirim şu komutla:
curl --request POST \
--url 'http://<nodejs-publicip>:5000/todos' \
--header 'content-type: application/json' \
--data '{"description":"DevOps-Betul-Kaplan-Ansible-Project"}'     #'{"description":"ansible project 207"}'
[{"todo_id":1,"description":"DevOps-Betul-Kaplan-Ansible-Project"}]  #[{"todo_id":1,"description":"ansible project 207"}]

bunu yapınca browserdan nodejs makine ip:5000/todos ile kontrol edersen girdiğin datanı görürsün.
bu işte json formatında bilgiler getiren bir API.
ŞİMDİ BUNA BİR ARAYÜZ KAZANDIRACAĞIZ .yani;

STEP-10:
 
react makineye geçiyoruz aynı şeyleri react makine için yapacağız.

1-önce env dosyasında nodejs makinemin public ip sini giriyorum. böyle frontend react makinem , backend nodejs makinemle bağlantı kurabiliyor.
private ip de girersem de todo sayfasını görürüm ama nodejs deki datalar publicip:5000 den yayın yaptığı için react arayüzde dataları göremem onlara ulaşamam.
yani bilgiler react a gelememiş olur.

2-şimdi react için Dockerfile yazıyoruz.
Dockerfile da geçen "yarn" de npm gibi bir paket maneger.
ama daha hızlı olduğundan daha popüler.cash leme yapabiliyor dolayısıyla daha önce yaptığı işlemleri
tekrar etmediğinden daha hızlı işlem yapmış oluyor.farklılık olsun diye bu sefer "npm" ile değil "yarn" kullandık.

3-src code ları makineye kopyaladık. Dockerfile ı yazdık . şimdi sıra 

4-react playbook

01-playbook dosyasına cd ile geç ve docker_react.yml oluştur.
02-aynı şekilde hosts ,src-kodlar ,path ,image ve container isimlerini tabiki değiştirdik.
03-şimdi playbook çalıştıralım .

-ansible-playbook docker_react.yml  

react makinenin public-ip:3000 den göreceğiz.

DİKKAT: bazen react makine image built ederken t2 micro olduğundan patlayabiliyor. 
bu yüzden makineyi stop ederek makine tipini yükseltebiliriz.önce ctrl+ c ile işlemi durdur.
react makineyi consola gidip stop et,sonra  Actions-instance settings-change instance type "t3a.medium" seç.
Apply tıkla. sonra react makineyi tekrar start et. makinenin hazır olmasını biraz bekle. 
şimdi yeniden playbook u çalıştır.
ansible-playbook docker_react.yml
çalıştırınca 
browserda react public-ip:3000 portundan uygulamamızın arayüznü görebiliriz artık

***************
***************
ALINAN HATALAR:
****************
****************
1-

ansible-vault kullanmıştık.secret.yaml dosyasına datamızı hatalı girmişiz.şöyle düzelttik
playbooks dosyası altındayken

ansible-vault edit secret.yml  # bu komutla secret.yml dosyamızı edit ettik.


dosya şu şekilde olmalı.
secret.yml içi:

password: Pp123456789 

bu şekilde vim dosyasını Esp + wq ile kaydedip çık.

tekrar çalıştır:

ansible-playbook --ask-vault-pass docker_postgre.yml

2- 

nodejs playbook 'da
docker_nodejs.yml de nodejs src code larını kopyalarken 

src : /home/ec2-user/ansible/ansible-project/nodejs/ # burada en sondaki slash unutulmuş.

öyle olunca altındaki dosyaları göremediğinden "cannot spacified Dockerfile "hatası aldım.


***ŞİMDİ BUNU TEK BİR PLAYBOOK OLARAK YAZSAK NASIL YAZARIZ***

1-playbooks dosyasına girip burada docker-project.yml oluştur.

2-hosts: burada terraformdosyası yazarken 3 makineyi de development şeklinde tag larda isim vermiştik.

3-dynemic inventory hazırlarken de _development olarak hosts ları gruplamıştık bu yüzden 

hosts: _development # bu şekilde playbook a girerim.

*** hepsine ortk olarak yapılacaklar

4-update etme

5-docker kurma

bu adımları ortak yapacağız .
-------
aşağıdakileri her makineye ayrı yapacağız:
6-kodları makineye gönderme 

7-var olan image ve container ları silme 

8-image build edip 

9-container çalıştırmayı .

10-değişkenleri yaz

burada da sadece 

* container path
* image adı 
* container adı

sadece bunlar değişeceği için;
bunları değişken olarak makinelerin playbooklarında yazarsam işlerim kolaylaşır
hem daha herkes tarafından kullanılabilir hale gelir.

ve playbook çalıştır.

-ansible-playbook --ask-vault-pass docker_project.yml

ANSİBLE İMPORT MODÜLÜ İLE PLAYBOOK YAZMA:
-

1-şimdi docker kuran playbook ile makinelere ayrı ayrı işlem yaptığım playbookları ayırıyorum.

2-playbooks altında install_docker.yml adında bir dosya açıp _development grubu altındaki 3 makineyi update edip docker kurma tasklarını buraya koyacağız. 3 makinenin playbook larında da tabi ozaman 
docker kurma kısımları yorum satırı olarak ayarlayacağız. çünkü update ve docker kurulumu için zaten elimde ayrı bir playbook var. 

3-şimdi Ansible import modülünü kullanarak bu 4 playbook u çalıştıran playbook u yazacağız.

playbooks altına "playbooks.yml" oluşturup içine, "ansible import" modülü ile bu 4 playbooku yazalım.
 yine çalıştırırken vault kullandığımız için :
 --ask-vault-pass # bu parametreyi unutma...!!!

 -ansible-playbook --ask-vault-pass playbooks.yml

### ROLE

 ROLE : ŞİRKETLERDE daha REUSEABLE BİR KULLANIM SAĞLIYOR.

--- 
 hazırlanmış olan rollerden playbook yeniden tkrar tekrar aynı playbookları yazmadan kullanabiliriz.
ANSİBLE GALAXY den de hazır ROLLER bulabiliriz.
---

şimdi bu playbookları bir role çevireceğiz. 
mesela docker yükleme şirket için sürekli yapılagelen bir işse bunu ROLE haline getirip tekrar tekrar yazmamış oluyoruz.

şöyle yaparız:
DİKKAT:!!!!!
"palybooks" dosyası içinde yapmayacağız.
"ansible-project" dosyasında da yapmayacağz .

1-bu iki dosyanın üstündeki "ansible" dosyasına çıkıp çalışacağız ROLE 'ü
ansible folder içinde iken :
mkdir roles && cd roles 

2-bu şekilde roles folder içine giriyoruz .Hatırlarsanız. 
"ansible_galaxy init" komutu ile içi boş bir dosya dizin geliyordu.
"ansible_galaxy install" komutu ile de hazır bir role ' ü kullanıyorduk.
kendimiz oluştururken "init", hazır role kullanırken "install".
biz burada kendimiz oluşturuyoruz.
update ve docker kurulum ortak taskları için bir role
nodejs,react, postgersql için ayrı role oluşturacağız.

01- docker kurulumu yapan role için:
roles dosyası içinde iken
- ansible_galaxy init docker # roles altında default bir docker folder oluştu.

02-
- ansible_galaxy init postgre

- ansible_galaxy init nodejs

- ansible_galaxy init react

hepsi için ayrı role folder larım oluştu  . burada hazır dosyalar var.

03-task folder altına task lerimi atacağım.

04-vars folder altına variable lerimi atacağım.

05-files folder altına src code ve Dockerfile ları atacağım. 

3- docker kurma için

01-şimdi oluşan "docker" altındaki default gelen "task" folder altındaki "main.yml" 'a docke kurulumu yapan tasklarımı kopyalayacağım.
istersem variable larımı vars altına gönderecektim ama bu bir docker kurulumu burada pek variable filan ya da 
files altına dosya filan atmam gereken bir task yok sadece update ve docker kurulumu yapıyor.

********* sırayla 3 makine için olan folderlara gerekli file ları atacağım.*********

4- postgre için:

01-oluşan postgre altındaki task altındaki main.yml 'a postgre nin task ını atıyorum.

02-files altına postgre için gerekli files atıyorum.

03-files altına ,direk ansible_project altındaki postgres(içinde Dockerfile ve initsql olan dosya) i direk copy-past yapıyorum.

04-postgre deki variable leri ve env yi , yani,
POSTGRES_PASSWORD: "{{password}}". oda secret.yaml içinde. o yüzden vars altındaki main.yml'a 

05-variablelerin altına bu env deki password ü de ekleyip . tüm dosyayı şifreleyeceğim.

# vars file for postgre

container_path: /home/ec2-user/postgresql 

container_name: cw_postgre

image_name: clarusway/postgre

password: Pp123456789  # bunu ekledim. 

***ANSİBLE DE DOSYA ŞİFRELEME***

06-şimdi bu tüm dosyayı şifreleyeceğim şöyle:

07-postgre nin roles altındaki vars dosyası altına (integrated terminal ile ) gir ve

-ansible-vault encrypt main.yml 

bu yukardaki komutla tüm dosyayı yani vars dosyamın içindeki main.yml yi şifrelemiş oldum.

tekrar decrypt etmek için de :

-ansible-vault decrypt  main.yml

5- şimdi nodejs için:

01-oluşan nodejs altındaki task altındaki main.yml 'a nodejs nin task ını atıyorum.

02-files altına ,direk ansible_project altındaki nodejs i direk copy-past yapıyorum.

03-nodejs playbook da variable leri vars altındaki main.yml içine yazıyorum.:
 onlar da 3 tane idi.

 ---
# vars file for nodejs
container_path: /home/ec2-user/nodejs

container_name: cw_nodejs

image_name: clarusway/nodejs

6- şimdi react için:

01-oluşan react altındaki task altındaki main.yml 'a react nin task ını atıyorum.

02-files altına ,direk ansible_project altındaki react i direk copy-past yapıyorum.

03-react playbook da variable leri vars altındaki main.yml içine yazıyorum.:
 onlar da 3 tane idi.

 ---
# vars file for react
container_path: /home/ec2-user/react
container_name: cw_react
image_name: clarusway/react

KISACASI :
ansible_galaxy init ile oluşan default dosyalardan 
ilgili olanına

task/main.yml 'a ilgili task ları kopyalıyoruz

files altına ilgili files 'ları kopyalıyoruz

variable (env) leri vars/main.yml içine yazıyorum.

7- şerkli dosyaları init ile gelen folderlar altına koyduktan sonra 
bu role leri çalıştıran playbook umu yazıyorum.

***ama dikkat bunu ansible altında açtığım playbooklarımın olduğu playbooks folder ın içinde yazacağım.***

playbooks folder içindeyken  
play_role.yml file oluşturup buna yazıyorum.

bunu yazarken roles altına rolün adını yazıyoruz sadece bu şekilde o rolü çağırıp 
onun altındaki dosyalarla playbook u çalıştıracak.
har makine için şu örnekteki gibi role ü çağıran playbook u yazacağız.

örnek:

- name: install docker 
  hosts: _deployment 
  become: true 
  roles:
    - docker

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
DİKKAT - DİKKAT!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!


8-bu şekli ile bu role playbook çalışmaz çünkü role leri nereden alacağını bilmiyor.
path ini göstermem gerekiyor. işte onu da ssh ile bağlı olduğum control node ec2-yser altına göndermiş olduğum
".ansible.cfg " içerisine ekliyorum.
şu şekilde bir dosya idi 

[defaults]
host_key_checking = False
inventory = inventory_aws_ec2.yml
deprecation_warnings=False
interpreter_python=auto_silent
private_key_file = /home/ec2-user/betulk.pem
remote_user=ec2-user
roles_path=/home/ec2-user/ansible/roles  # you must show the roles path


en sondaki;

roles_path=/home/ec2-user/ansible/roles 

ekledik.çünkü roles dosyam ansible diye /home/ec2-user altında oluşturduğum dosyanın altında oluşturmuştum.

ROLE için
ansible_galaxy init ile 4 role oluşturduk
gerekli tasks, files ve vars ları yerine koyduk.
ansible.cfg içine roles_path ekledik.
password olan var dosyamı ansible_vault encrypt ile şifreledik.
bu rolleri çağımak için play_role.yml adında playbook 'umuzu yazdık.

9-ŞİMDİ ÇALIŞTIR
yine playbooks dosyamın altına gidip

-ansible_playbook --ask-vault-pass play_role.yml 

evet bu yukardaki komutla çalıştır.
react-publicip:3000 portundan da bak

ChatGPT
PROJEYE HAZIRLIK ADIMLARI
STEP-1:
Terraform dosyalarını çalıştır. 4 makine oluşur: postgresql, nodejs, react, control-nodejs.

STEP-2:
Control-node'a SSH ile bağlan ve aşağıdaki komutları çalıştır:

bash
Copy code
ansible all -m ping # Makinelere ping atabiliyorsak çalışmaya başlayabiliriz.
ansible all -m ping -o # Tek satır halinde makinelerden pong döner.
STEP-3:
Eğer SSH ile bağlandığın control node'a "ansible.cfg", "inventory_aws_ec2.yaml" dosyası ve .pem uzantısı ile beraber pem-key dosyan geldiyse tamamdır. Not: Dikkat, key ".pem" uzantısı ile geldiğine emin ol. Bazen KEY gelmeyebiliyor.

STEP-4:
VSCode Ansible extension yükleyelim.

STEP-5:
PROJENİN RESİMDEN AÇIKLAMASI:

Bu uygulama Nodejs uygulaması. PERN diye de kısaltması geçer:

PostgreSQL
ExpressJS
React
NodeJS
MERN kısaltması ile de geçer. PostgreSQL yerine MongoDB kullanıyorsak MERN denir.

PROJE NIN RESMINE BAKARSAK:

CONTROL NODE:

Bir tane control node var, bunu zaten Terraform ile kurduk. Terraform ile kurarken EC2 FULL access yetkisini vermiştik. Dynamic inventory kullanıyorsak:

EC2 full access
Boto3
Control node'da:

Config file
Inventory
PEM file
Playbook da biz yazacağız.
3 makine daha kurulmuştu Terraform ile:

PostgreSQL: Todo uygulamasının datalarını tutuyor.
NodeJS: Backend kısmı, datayı oluşturup PostgreSQL'e aktarıyor. JSON formatında bir API.
React: Frontend, todo arayüzünü gösteriyor.
NOT: Bu 3 makineyi tek makine de toplayabilirdik Docker Compose ile de yazabilirdik. Ama burada Ansible ile configure edilmeyi örnekliyoruz.

3 makinenin her biri bir container ama her biri ayrı bir EC2'ya kuruluyor. Ansible ile configureyi anlayabilmemiz için.

Code dosyalarındaki:

client = react
database = postgresql
server = nodejs
NOT: Control node t3a.medium, diğer 3 makine t2.micro. DİKKAT!!!! Bazen React makine imajları oluştururken sıkıntı çıkarabiliyor. O zaman gidip React makine tipini yükseltmek gerekebilir. O zaman makineyi stop edip makinenin tipini değiştirebiliriz.

STEP-6:
Source code'ları yükle, Dockerfile'larını yaz ve imajlarını oluştur. Source code'lar developerlar tarafından yazılmış. Biz bunların dosyalarını control-node yükleyip Dockerfile'larını yazacağız.

STEP-7:
İmajlar hazır olduktan sonra playbook'ları yazacağız. Her bir makine için ayrı playbook'lar yazacağız.

STEP-8:
PROJENIN TASK INI README'DEN İNCELERSEK:

Portlar zaten hazır. Terraform'la kurarken zaten açtık:

NodeJS: 5000
PostgreSQL: 5432
React: 3000
Source code'lar ilgili makinelere gönderilecek.

Bütün node'lar control-node'dan yönetilecek.

Dynamic inventory kullanıyoruz.

Ansible config file hazır.

Docker ve Ansible kullanılarak tüm makinelere kurulmalı.

PostgreSQL:

Source code'ları çekip Dockerfile'ları yazıp imajları oluşturacağız. Önce PostgreSQL için yazacağız. Tabi init.sql bize verilmişti zaten, o da dahil edilmeli.
PostgreSQL için gerekli env'leri Ansible Vault ile oluşturup, PostgreSQL container'ı ayağa kaldıracağız.
Security group'lar zaten Terraform'la ayarlanmış oldu.
Container kapanınca herhangi bir nedenle bilgiler gitmemesi için volume bağlayacağız.
NodeJS Worker Node: Backend

Dataları oluşturup PostgreSQL'e aktaracak. Bu yüzden PostgreSQL'in private IP'sini NodeJS'in env dosyasına girmem gerekir. Aynı şekilde React da NodeJS ile konuşabilmesi için: NodeJS public IP'sini React'in env dosyasına girmem lazım.
İmaj oluştur.
Port 5000 zaten açık (Terraform).
Security group (Terraform) ayarlanmış.
React Worker Node: Frontend

Env file'a NodeJS public IP'sini yazacağız.
Source code'ları gönder, Dockerfile yaz, imaj hazırla.
Port: 3000 - 80 zaten (Terraform).
Security group zaten (Terraform). Uygulamayı 3000 portundan görürüz.
Control Node için:

Installing Ansible
ansible.cfg
inventory_aws_ec2.yaml (Dynamic-inventory). Dynamic-inventory dosyası mutlaka şu şekilde biter "_aws_ec2.yaml".
Bu 3 gerekli olmazsa olmazı Terraform ile kurduk.

STEP-9:
3 tane Dockerfile, 3 tane Playbook yazacağız.

PLAYBOOK'LAR:
PostgreSQL için:
Playbook'da şunlar var:

Update
Install Docker
init.sql kopyalama
Dockerfile kopyalama
İmaj oluşturma
Container çalıştırma
Bazı ekstra işlemler (eski container ve imajları silme gibi)
Özetle:

Docker kurulumu
init.sql yani source code'u önce Github'dan control node'a alacağız, kopyalayacağız. Sonra buradan ilgili server'a playbook yazarak göndereceğiz. Copy module kullanacağız.
Önceden yazdığım Dockerfile'ı playbook'da copy modulünü kullanarak ilgili server'a gönder.
Playbook'da imaj oluşturacağız.
Playbook'da container çalıştıracağız.
ÖZET-ÖZET-ÖZET

Özetle tüm yapacağımız iş şu:

Control node'da PROJE folder oluştur.
Onun altında 3 adet folder oluştur: postgresql, nodejs, react
Bu folder'lara ilgili source code'ları koyacağız. Dockerfile'ı orada yazacağız. İlgili server'a gönder.
İlgili server'da önce docker kuracağız.
İlgili dosyaları control node'dan ilgili server'a gönder.
İmaj oluştur.
Container çalıştır.
NodeJS:
Playbook için:

Update
Install Docker
Server files kopyalama
Dockerfile kopyalama
İmaj oluşturma
Container çalıştırma
Bazı ekstra işlemler
React:
Playbook için:

Update
Install Docker
Client files kopyalama
Dockerfile kopyalama
İmaj oluşturma
Container çalıştırma
Bazı ekstra işlemler
Extra işlemler:

Env dosyasındaki "ip"lerin değiştirilmesi gibi
INVENTORY DOSYASI HAKKINDA
Filter'lar ve key_group'lar kullandık. Doğru host'lara ulaşmak için doğru filter yapmalıyım. Gruplamaları yaparken kullandığım tag'leri vermeyi Terraform yazarken düşünmeliyiz.

PROJE ADIMLARI
Şimdi projeye başlayalım:

STEP-1:
Control-node'a bağlan ve ana-dizinde:

```bash
Copy code
mkdir ansible
cd ansible
mkdir ansible-project playbooks
cd ansible-project
mkdir postgres nodejs react
```

$ ls -R   # Dosya yapısının ayrıntısını verir.

-- Ya da tree'yi indirip görebilirsin:

$ sudo dnf install tree -y

STEP-2:

Şimdi source code'ları bu gerekli folder'lara kopyalayacağız. Dockerfile'ları da burada yazacağız. Daha sonra playbook'ları yazarken bu dosyaları buradan alıp gerekli server'ların içerisine Ansible ile göndereceğiz. Playbook'ları da playbooks dosyası altında yazacağız. Daha rahat çalışmak için oluşturduğum ansible dosyasına geçebilirim, solda dosya yapısı karışık olmasın istersem.

Postgres:

- Source-code
- Dockerfile
- student_files altındaki todo-app-pern dosyasının içindeki database altındaki  init.sql'i oluşturduğum ansible altındaki postgresql dosyasına sürükle at.

## Şimdi postgres altına (newFile) Dockerfile'ını yazacağız.

STEP-3:

*** Postgres Dockerfile'ı ***

- dockerfile
- Copy code
- FROM postgres
- COPY ./init.sql /docker-entrypoint-initdb.d/  # Dockerhub postgres image açıklamasından buldum (initialization scripts)
- EXPOSE 5432 # Yazılmasa da olur bu sadece bilgilendirmedir.

STEP-4:

Playbook for postgres:

Dikkat edilecek hususlar:

- hosts: Bu playbook'un çalıştırılacağı host'ları belirler.
- vars: Bu playbook'a özgü değişkenler. Bu örnekte, db_name, db_user, db_password gibi PostgreSQL container'ı için gerekli ortam değişkenlerini belirler.
- tasks: Bu playbook'un çalıştıracağı görevler.
- name: Görevin adını belirtir.
- package: Belirtilen paket yöneticisini kullanarak paketlerin yüklenmesini sağlar. Bu örnekte docker paketi yüklenir.
- copy: Dosyaların belirtilen kaynak yolundan hedef yola kopyalanmasını sağlar. Bu örnekte Dockerfile ve init.sql dosyaları kopyalanır.
- command: Belirtilen komutun çalıştırılmasını sağlar. Bu örnekte Docker imajı oluşturulur ve container çalıştırılır.

- hosts: postgres #  Bu playbook'un PostgreSQL sunucusunda çalıştırılacağını belirtir.
- vars: PostgreSQL container'ı için gerekli ortam değişkenlerini tanımlar.
- tasks:
Docker'ın kurulu olduğundan emin olur.
init.sql ve Dockerfile dosyalarını ilgili sunucuya kopyalar.
Docker imajı oluşturur ve container'ı çalıştırır.
PostgreSQL container'ının çalıştığını doğrular.

NOT:

Bütün işlemler bu mantıkla olacak. NodeJS ve React için de benzer şekilde Dockerfile'ları yazıp imaj oluşturacağız. Daha sonra playbook'larla bu dosyaları ilgili sunuculara gönderecek ve container'ları çalıştıracağız.

Bu playbook'lar çalıştırıldığında:

PostgreSQL container'ı başlatılacak ve init.sql çalışacak.
NodeJS container'ı PostgreSQL ile bağlantılı olarak başlatılacak.
React container'ı NodeJS backend ile bağlantılı olarak başlatılacak.
Tüm playbook'ları çalıştırarak uygulamanın tüm bileşenlerini ayağa kaldırabiliriz.