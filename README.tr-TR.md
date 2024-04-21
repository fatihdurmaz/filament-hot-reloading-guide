# Filament - Vite Hot Reloading

Geliştiriciler için zaman ve verimlilik büyük önem taşır. Vite, modern web uygulamaları için hızlı bir geliştirme sunan bir araçtır. Hot reloading, uygulama üzerinde yapılan değişiklikleri hızlı bir şekilde görüntülemek için kullanılan güçlü bir özelliktir.

Bu Rehber, Laravel Filament kullanarak bir web uygulaması geliştirirken Vite aracını nasıl entegre edeceğinizi ve hot reloading özelliğini nasıl etkinleştireceğinizi göstermektedir.

## 1. Laravel Vite Eklentisi
### a. vite.config.js Hot Reloading Entegrasyonu (Opsiyonel)

*Bu eklenti her değişiklik yapıldığında sayfayı yeniler. Değişiklik yapılan bileşen veya sayfadaki herhangi bir elementin kendisini yenilemez.*
        
    import { defineConfig } from 'vite';
    import laravel, { refreshPaths } from 'laravel-vite-plugin'
    
    export default defineConfig({
        plugins: [
            laravel({
                input: ['resources/css/app.css', 'resources/js/app.js'],
                refresh: [
                    ...refreshPaths,
                    'app/Http/Livewire/**', // Custom Livewire components
                    'app/Filament/**', // Filament Resources
                ],
            }),
        ],
    });
        
    
### b. Vite Livewire Eklentisi (Opsiyonel - Önerilen)
#### i. [vite-livewire-plugin](https://github.com/defstudio/vite-livewire-plugin) eklentisinin kurulması ve app.js dosyasında çağrılması

*Bu eklenti her değişiklik yapıldığında bileşen veya sayfadaki herhangi bir elementin kendisini yeniler. Sayfanın tamamının yenilenmesine gerek duymaz.*
        
    npm install --save-dev @defstudio/vite-livewire-plugin
    
    // resources/js/app.js
    
    import './bootstrap';
    import { livewire_hot_reload } from 'virtual:livewire-hot-reload'
    livewire_hot_reload();
        
#### ii.vite.config.js Hot Reloading Entegrasyonu
        
    import { defineConfig } from 'vite';
    import laravel, { refreshPaths } from 'laravel-vite-plugin';
    import livewire from '@defstudio/vite-livewire-plugin'; // yüklediğimiz eklentiyi import ediyoruz.
    
    export default defineConfig({
        plugins: [
            laravel({
                input: [
                    'resources/css/app.css',
                    'resources/js/app.js',
                ],
                refresh: false,
            }),
    
            livewire({
                refresh: [
                    ...refreshPaths, // app.js çağrıldığı her yerdeki değişiklikler izleniyor
                    'app/Http/Livewire/**', // LiveWire bileşenlerini izlemek için (varsa)
                    'app/Custom/Path/**', // vite aracının takip etmesini istediğiniz dosyaların yolunu gösterebilirsiniz.
                ]
            }),
        ],
    })
        
    
## 2.Filament Entegrasyonu

*Değişikliklerin Vite tarafından takip edilebilmesi için app.js dosyasının Filament paneline eklenmesi gereklidir. 2 şekilde yapabiliriz.*
        
### a. AppServiceProvider → register() yönteminde
            
    use Filament\Support\Facades\FilamentView;
    use Illuminate\Support\Facades\Blade;
    
    public function register()
    {
        FilamentView::registerRenderHook('panels::body.end', fn(): string => Blade::render("@vite('resources/js/app.js')"));
    }
            
### b. AdminPanelProvider → renderHook() yönteminde
            
    <?php
    
    namespace App\Providers\Filament;
    ...
    ...
    use Illuminate\Support\Facades\Blade;
    
    class AdminPanelProvider extends PanelProvider
    {
        public function panel(Panel $panel): Panel
        {
            return $panel
                ->default()
                ->id('admin')
                ->login()
                ->colors([
                    'primary' => Color::Amber,
                ])
                ...
                ...
                ->collapsibleNavigationGroups(false)
                ->renderHook('panels::body.end', fn (): string => Blade::render("@vite('resources/js/app.js')"))
                            ...
                            ...
                            ...
        }
    }            

> Yukarıdaki adımlar tamamlandıktan sonra Vite aracını ve Laravel artisan sunucusunu çalıştırarak hot reloading özelliğin aktif edildiğini görmüş olacağız.
> 

```bash
npm run dev // vite sunucusunun çalıştırılması
php artisan serve // Laravel artisan sunucusunun çalıştırılması
```

<aside>
💡 Herhangi bir sorunla karşılaşmadığınız halde hot reloading çalışmıyorsa `php artisan optimize` komutunu çalıştırarak Laravel uygulamasını optimize edip sorunu çözebilirsiniz.

</aside>

