# CCTDD: Cypress Bileşen Test Odaklı Tasarım

## Önsöz

Cypress Bileşen Testi ile TDD (Test Driven Design) uygulamanın, mühendisliğimizi bir üst seviyeye taşıyabileceğine inanıyorum.

Bu kitapta Angular'ın Kahramanlar Turu'nu (Tour of Heroes), Cypress, React ve TS kullanarak bileşen test odaklı bir şekilde yeniden oluşturacağız. Bileşenleri oluşturduktan sonra, TDD'nin yönlendirme, durum yönetimi ve Cypress e2e testi ile nasıl uygulanabileceğini inceleyeceğiz. Ayrıca Cypress ile API e2e testi hakkında bilgi edineceğiz ve UI entegrasyon testlerinin nerede yer alacağını göreceğiz.

Gereksinim özellikleri için [uygulamanın Angular versiyonunu](https://papa-heroes-angular.azurewebsites.net/heroes) kullanacağız. Kitabın ilk yarısında, bileşenlerimizi mühendislik için Cypress bileşen testi ile test odaklı tasarım yaklaşımını kullanacağız. Önce alt bileşenler üzerinde çalışacağız ve yol kat edeceğiz. Tıklama olayı işleyicileri için karar vermemiz gereken zamana kadar `console.log` kullanacağız.

Daha sonraki bölümlerde durum ve yönlendirmeye değindiğimizde, Cypress e2e testi TDD için daha alakalı hale gelecek ve bileşen test paketi, uygulamamızın hala beklendiği gibi çalıştığı konusunda bize güvence sağlayacaktır.

Her bölümde TDD'nin Kırmızı Yeşil Refaktör döngülerini açıkça yakalayacak ve Cypress bileşen testinin React Testing Library'ye (RTL) 1:1 kıyaslaması ile bir özet ve anahtar noktalarla sona ereceğiz. Her kod örneği, kopyalanabilir ve tekrarlanabilir sonuçlar vermek için tasarlanmıştır, ancak okuyucunun kodu kopyalamak yerine yazıp kademeli olarak iyileştirmesi teşvik edilir. Son bir uyarı; erken bölümlerde, dosyaları ve kodları daha sonra büyük ölçüde yeniden düzenlemekten kaçınmak için, uygulamanın son sürümüyle daha uyumlu olacak şekilde bazı kararlar alabiliriz.

Yıllar önce John Papa, Angular, Vue ve ReactJS'de aynı uygulamayı oluşturdu, ne yazık ki React versiyonu eski ve React topluluğu olarak meta değişikliklerinin ne sıklıkta yapıldığını biliyoruz. Yaklaşım, Vue ve Angular'dan gelen okuyuculara Cypress bileşen testleri yazarken 1:1 karşılaştırmalar yapma ve hatta bu çerçevelerden birinde bu alıştırmayı yeniden yapma olanağı sunar.

## Teşekkürler

Bu içeriğin gözden geçirenlerine çok teşekkürler; [Matthew Schrepel](https://www.linkedin.com/in/mschrepel/), [Stefano Magni](https://www.linkedin.com/in/noriste/), [Gleb Bahmutov](https://www.linkedin.com/in/bahmutov/) ve [Kent C. Dodds](https://www.linkedin.com/in/kentcdodds/) sürekli olarak Epic React ofis saatleri boyunca soruları yanıtladığı için.

## Bağlantılar ve Kaynaklar

Bu kitabın içeriğine ilham veren kaynaklar şunlardır:

[React Hooks in Action - John Larsen](https://www.manning.com/books/react-hooks-in-action)

[Epic React - Kent C. Dodds](https://epicreact.dev/)

[Learning TDD - Saleem Siddiqui](https://www.oreilly.com/library/view/learning-test-driven-development/9781098106461/)

[Tour of Heroes - John Papa](https://papa-heroes-angular.azurewebsites.net/heroes)

## Cypress Bileşen + e2e örnekleri bulunan GitHub depoları

[tour-of-heroes-react-cypress-ts - Bu kitapta CRA & Webpack kullanılarak inşa edilen son uygulama](https://github.com/muratkeremozcan/tour-of-heroes-react-cypress-ts) & [aynı uygulama Vite ile](https://github.com/muratkeremozcan/tour-of-heroes-react-vite-cypress-ts)

[react-hooks-in-action-with-cypress - React Hooks in Action kitabında inşa edilen son uygulama](https://github.com/muratkeremozcan/react-hooks-in-action-with-cypress)

[bookshelf - Epic React'tan son uygulama](https://github.com/muratkeremozcan/bookshelf)

[365+ Cypress bileşen testi örneği](https://github.com/muratkeremozcan/cypress-react-component-test-examples)

## Gelecek

Karşılaştırma ve zıtlıklar yapmak, yeni teknolojileri öğrenmenin harika bir yoludur. Bu tarzda yapılan çok fazla kaynak yoktur. Örneğin bu içerikte Kahramanlar, bileşenler arasında durumu iletmek için React özelliklerini kullanırken, benzerleri olan Kötüler ise React context API'sini kullanmaktadır.

Bu, günlük olarak güncellenen ve geliştirilen canlı bir kitaptır. Redux aynası, Redux Toolkit aynası, X-State aynası ekleyebiliriz, belki bir gün uygulamayı NextJs'e taşıyabiliriz. Gelecekte daha fazla bölüm görmeyi ve birlikte ön uç geliştirme hakkında daha fazla bilgi edinmeyi bekleyin.

## Sorular, Yorumlar ve Geri Bildirimler

Yukarıdaki tüm konular için [tartışma](https://github.com/muratkeremozcan/cctdd-tr/discussions) başlatabilir veya [CCTDD GitHub deposunda](https://github.com/muratkeremozcan/cctdd-tr) bir [sorun](https://github.com/muratkeremozcan/cctdd-tr/issues) açabilirsiniz.

## Kurulum

Yeni bir depo oluşturmak için bu [şablonu](https://github.com/muratkeremozcan/react-cypress-ts-template) kullanın. Depo, React, TS, Cypress (e2e & ct), GHA ile CI mimarisi, Jest, ESLint, Prettier, Renovate, Husky, Lint-staged ve yeni bir projeye başlamadan önce sahip olmak istediğiniz çoğu şeyi içerecektir. Yeni depoyu klonlayın ve yükleyin.

Talimatlar için, oluşturduğunuz depoyu https://github.com/muratkeremozcan/tour-of-heroes-react-cypress-ts olarak kabul edeceğiz, **burada uygulamanın son sürümünü bulabilirsiniz**. Örnekler `yarn` kullanır ve tercihe bağlı olarak `npm` kullanılabilir.

```bash
git clone https://github.com/muratkeremozcan/tour-of-heroes-react-cypress-ts
cd tour-of-heroes-react-cypress-ts
# eğer şirket vekil sunucusu üzerinde kurulum yapıyorsanız --registry değiştiricisini ekleyin
# aksi takdirde sadece yarn install
yarn install --registry https://registry.yarnpkg.com
# not: buradan itibaren --registry https://registry.yarnpkg.com
# şirket vekil sunucusu üzerinde çalışıyorsanız eklenmesi gerektiğini varsayın
# aksi takdirde depoyu şirketin özel GitHub alanında barındırmak zorunda kalacaksınız
```

Bir sağlamlık kontrolü için, komutları teker teker çalıştıracağız.

```bash
# paralel ünite, tip kontrol, düzeltme, biçimlendirme, derleme
yarn validate

# e2e, ct ve ünite testlerini çalıştırın

# Cypress bileşen test yürütücüsünü kontrol edin
yarn cy:open-ct

# Cypress e2e test yürütücüsünü kontrol edin (uygulamayı başlatır)
yarn cy:open-e2e

# Jest test yürütücüsünü yerel olarak izleme modunda başlatır
yarn test
```
# 
