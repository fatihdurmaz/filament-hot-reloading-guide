# Filament - Vite Hot Reloading

For developers, time and productivity are of great importance. Vite is a tool that offers fast development for modern web applications. Hot reloading is a powerful feature used to quickly view changes made to an application.

This guide demonstrates how to integrate the Vite tool and enable the hot reloading feature when developing a web application using Laravel Filament.

## 1. Laravel Vite Plugin

### a. vite.config.js Hot Reloading Integration (Optional) 

*This plugin refreshes the page every time a change is made. It does not refresh the modified component or any element on the page.*


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
        
### b. Vite Livewire Plugin (Optional)
*This plugin refreshes the component or any element on the page every time a change is made. There is no need to refresh the entire page.*

#### i.[vite-livewire-plugin](https://github.com/defstudio/vite-livewire-plugin) installing the plugin and calling it in the app.js file (Recommented)
        
           npm install --save-dev @defstudio/vite-livewire-plugin

           // resources/js/app.js
           
           import './bootstrap';
           import { livewire_hot_reload } from 'virtual:livewire-hot-reload'
           livewire_hot_reload();
        
#### ii. vite.config.js Hot Reloading Integration
        
        import { defineConfig } from 'vite';
        import laravel, { refreshPaths } from 'laravel-vite-plugin';
        import livewire from '@defstudio/vite-livewire-plugin'; // import plugin
        
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
                        ...refreshPaths, // Tracking changes wherever app.js is called
                        'app/Http/Livewire/**', // To monitor LiveWire components (if applicable)
                        'app/Custom/Path/**', // You can show the path to the files you want the vite tool to follow.
                    ]
                }),
            ],
        })
        
## 2. Filament Integration

*In order for the changes to be followed by Vite, the app.js file must be added to the Filament panel. We can do it in 2 ways.*
   
### a. AppServiceProvider → register() function
            
            use Filament\Support\Facades\FilamentView;
            use Illuminate\Support\Facades\Blade;
            
            public function register()
            {
                FilamentView::registerRenderHook('panels::body.end', fn(): string => Blade::render("@vite('resources/js/app.js')"));
            }
            
### b. AdminPanelProvider → renderHook() function
            
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

> After the above steps are completed, we will see that the hot reloading feature has been activated by running the Vite tool and Laravel artisan server.

        npm run dev // vite server run
        php artisan serve // Laravel artisan server run

<aside>
💡 If you do not encounter any problems but hot reloading is not working, you can optimize the Laravel application and solve the problem by running the 'php artisan optimize' command.
</aside>
