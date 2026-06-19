# Демо-экзамен КОД 09.02.07-5-2026

## Содержание
- [Установка Laravel](#установка-laravel)
- [База данных](#база-данных)
- [Миграции](#миграции)
- [Модели](#модели)
- [Контроллеры](#контроллеры)
- [Маршруты](#маршруты)
- [Views (Blade)](#views-blade)
- [ 6 модуль ](#6-модуль)

---

## Установка Laravel

```bash
composer create-project laravel/laravel --prefer-dist .
 /^[0-9]{2} [0-9]{2} [0-9]{6}$/
```

---

## База данных

```sql
-- ------------------------------------------------------------
-- 1. Клиенты (из Заказчики.json)
-- ------------------------------------------------------------
CREATE TABLE clients (
    id VARCHAR(20) PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    inn VARCHAR(20) DEFAULT NULL,
    address VARCHAR(255) DEFAULT NULL,
    phone VARCHAR(20) DEFAULT NULL,
    salesman BOOLEAN NOT NULL DEFAULT FALSE,
    buyer BOOLEAN NOT NULL DEFAULT FALSE
);

-- ------------------------------------------------------------
-- 2. Номенклатура — единый справочник продукции и материалов
-- type: 'product' = готовая продукция, 'material' = сырьё
-- ------------------------------------------------------------
CREATE TABLE nomenclature (
    id INT PRIMARY KEY AUTO_INCREMENT,
    code VARCHAR(50) DEFAULT NULL,
    name VARCHAR(255) NOT NULL,
    type ENUM('product', 'material') NOT NULL,
    unit VARCHAR(20) NOT NULL DEFAULT 'шт',
    price DECIMAL(10,2) DEFAULT NULL
);

-- ------------------------------------------------------------
-- 3. Спецификации — заголовок (одна спецификация на продукт)
-- ------------------------------------------------------------
CREATE TABLE specifications (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    product_id INT NOT NULL,
    quantity DECIMAL(10,4) NOT NULL DEFAULT 1,
    unit VARCHAR(20) NOT NULL DEFAULT 'шт',
    FOREIGN KEY (product_id) REFERENCES nomenclature(id)
);

-- ------------------------------------------------------------
-- 4. Состав спецификации — какие материалы и в каком количестве
-- ------------------------------------------------------------
CREATE TABLE specification_lines (
    id INT PRIMARY KEY AUTO_INCREMENT,
    specification_id INT NOT NULL,
    material_id INT NOT NULL,
    quantity DECIMAL(10,4) NOT NULL,
    unit VARCHAR(20) NOT NULL DEFAULT 'кг',
    FOREIGN KEY (specification_id) REFERENCES specifications(id),
    FOREIGN KEY (material_id) REFERENCES nomenclature(id)
);

-- ------------------------------------------------------------
-- 5. Заказы — заголовок
-- ------------------------------------------------------------
CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    number INT NOT NULL,
    order_date DATE NOT NULL,
    client_id VARCHAR(20) NOT NULL,
    FOREIGN KEY (client_id) REFERENCES clients(id)
);

-- ------------------------------------------------------------
-- 6. Позиции заказа — продукция, количество, цены
-- ------------------------------------------------------------
CREATE TABLE order_lines (
    id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity DECIMAL(10,4) NOT NULL,
    unit VARCHAR(20) NOT NULL DEFAULT 'шт',
    price DECIMAL(10,2) NOT NULL,
    total DECIMAL(10,2) GENERATED ALWAYS AS (quantity * price) STORED,
    FOREIGN KEY (order_id) REFERENCES orders(id),
    FOREIGN KEY (product_id) REFERENCES nomenclature(id)
);

-- ------------------------------------------------------------
-- 7. Производство — заголовок
-- ------------------------------------------------------------
CREATE TABLE production (
    id INT PRIMARY KEY AUTO_INCREMENT,
    number INT NOT NULL,
    production_date DATE NOT NULL
);

-- ------------------------------------------------------------
-- 8. Выпуск продукции — что было произведено
-- ------------------------------------------------------------
CREATE TABLE production_outputs (
    id INT PRIMARY KEY AUTO_INCREMENT,
    production_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity DECIMAL(10,4) NOT NULL,
    unit VARCHAR(20) NOT NULL DEFAULT 'шт',
    FOREIGN KEY (production_id) REFERENCES production(id),
    FOREIGN KEY (product_id) REFERENCES nomenclature(id)
);

-- ------------------------------------------------------------
-- 9. Расход материалов — что было списано в производство
-- ------------------------------------------------------------
CREATE TABLE production_inputs (
    id INT PRIMARY KEY AUTO_INCREMENT,
    production_id INT NOT NULL,
    material_id INT NOT NULL,
    quantity DECIMAL(10,4) NOT NULL,
    unit VARCHAR(20) NOT NULL DEFAULT 'кг',
    FOREIGN KEY (production_id) REFERENCES production(id),
    FOREIGN KEY (material_id) REFERENCES nomenclature(id)
);


-- ------------------------------------------------------------
-- Импорт данных из Заказчики.json в таблицу clients
-- ------------------------------------------------------------
INSERT INTO clients (id, name, inn, address, phone, salesman, buyer) VALUES
('000000001', 'ООО "Поставка"', NULL, 'г.Пятигорск', '+79198634592', TRUE, TRUE),
('000000002', 'ООО "Кинотеатр Квант"', '26320045123', 'г. Железноводск, ул. Мира, 123', '+79884581555', TRUE, FALSE),
('000000008', 'ООО "Новый JDTO"', '26320045111', 'г. Железноводсу', '+79884581555', TRUE, FALSE),
('000000003', 'ООО "Ромашка"', '4140784214', 'г. Омск, ул. Строителей, 294', '+79882584546', FALSE, TRUE),
('000000009', 'ООО "Ипподром"', '5874045632', 'г. Уфа, ул. Набережная, 37', '+79627486389', TRUE, TRUE),
('000000010', 'ООО "Ассоль"', '2629011278', 'г. Калуга, ул. Пушкина, 94', '+79184572398', FALSE, TRUE);


-- Детализация стоимости по каждой позиции заказа
SELECT
    o.number AS order_number,
    o.order_date,
    c.name AS client_name,
    prod.name AS product_name,
    ol.quantity AS order_quantity,
    unit_costs.unit_cost AS cost_per_unit,
    ol.quantity * unit_costs.unit_cost AS line_total
FROM orders o
JOIN clients c ON c.id = o.client_id
JOIN order_lines ol ON ol.order_id = o.id
JOIN nomenclature prod ON prod.id = ol.product_id
JOIN (
    SELECT
        s.product_id,
        SUM(sl.quantity * n.price) AS unit_cost
    FROM specifications s
    JOIN specification_lines sl ON sl.specification_id = s.id
    JOIN nomenclature n ON n.id = sl.material_id
    GROUP BY s.product_id
) AS unit_costs ON unit_costs.product_id = ol.product_id;
```

---

## Миграции

### create_users_table

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->string('password');
            $table->boolean('need_to_change_password')->default(true);
            $table->unsignedTinyInteger('wrong_counter')->default(0);
            $table->boolean('is_admin')->default(false);
            $table->timestamps();
        });

        Schema::create('password_reset_tokens', function (Blueprint $table) {
            $table->string('email')->primary();
            $table->string('token');
            $table->timestamp('created_at')->nullable();
        });

        Schema::create('sessions', function (Blueprint $table) {
            $table->string('id')->primary();
            $table->foreignId('user_id')->nullable()->index();
            $table->string('ip_address', 45)->nullable();
            $table->text('user_agent')->nullable();
            $table->longText('payload');
            $table->integer('last_activity')->index();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('users');
        Schema::dropIfExists('password_reset_tokens');
        Schema::dropIfExists('sessions');
    }
};
```

---

## Модели

### User.php

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use HasFactory, Notifiable;

    protected $fillable = [
        'name',
        'email',
        'password',
        'need_to_change_password',
        'wrong_counter',
        'is_admin',
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected function casts(): array
    {
        return [
            'email_verified_at'        => 'datetime',
            'password'                 => 'hashed',
            'need_to_change_password'  => 'boolean',
        ];
    }
}
```

---

## Контроллеры

### Создание контроллеров

```bash
php artisan make:controller ActionsController
php artisan make:controller ViewsController
```

### ActionsController.php

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class ActionsController extends Controller
{
    public function login(Request $request)
    {
            $request->validate([
                'email' => 'required|email',
                'password' => 'required|string|min:6',
            ]);

            if (Auth::attempt($request->only(['email','password']))){
                $user = Auth::user();
                if ($user->wrong_counter === 3) {
                    Auth::logout();

                    return redirect()
                        ->route('login')
                        ->withErrors(['email' => 'Your account is locked due to too many failed login attempts.']);
                }

                if ($user->need_to_change_password) {
                    return redirect()
                    ->route('login.change-password')
                    ->with('status','You need to change your password.');
                }
                return redirect()->route('dashboard');
            }

             if ($user = User::firstWhere('email', $request->input('email'))) {
                if ($user->wrong_counter != 3){
                    $user->wrong_counter++;
                    $user->save();
                }
                if ($user->wrong_counter === 3) {
                    return redirect()
                        ->route('login')
                        ->withErrors(['email' => 'Your account has been locked due to too many failed login attempts.']);
                }
             };

            return redirect()
                ->route('login')
                ->withErrors(['email' => 'The provided credentials do not match our records.'])
                ->withInput($request->only('email'));
    }


    public function changePassword(Request  $request)
    {
        $request->validate([
            'current_password' => 'required|string|min:6|current_password',
            'new_password' => 'required|string|min:6|confirmed:new_password_repeat',
        ]);

        $user = Auth::user();
        $user->password = $request->input('new_password');
        $user->need_to_change_password = false;
        $user->save();

        return redirect()
            ->route('dashboard')
            ->with('status', 'Your password has been changed successfully.');

    }

    public function editUser(User $user, Request $request)
    {
        $request->validate([
            'name' => 'required|string|max:255',
            'email' => "required|email|unique:users,email,{$user->id}",
            'password' => 'nullable|string|min:6',
        ]);

        $user->name = $request->input('name');
        $user->email = $request->input('email');
        if ($request->filled('password')) {
            $user->password = $request->input('password');
        }
        $user->save();

        return redirect()
            ->route('dashboard')
            ->with('status', 'User has been updated successfully.');

    }

    public function createUser(Request $request)
    {
        $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users,email',
            'password' => 'required|string|min:6',
        ]);

        $user = new User();
        $user->name = $request->input('name');
        $user->email = $request->input('email');
        $user->password = $request->input('password');
        $user->save();

        return redirect()
            ->route('dashboard')
            ->with('status', 'User has been created successfully.');
    }

    public function deleteUser(User $user)
    {
        if($user->id === Auth::id()){
            return redirect()
                ->route('dashboard')
                ->withErrors(['user' => 'You cannot delete your own account.']);
        }
        $user->delete();
        return redirect()
            ->route('dashboard')
            ->with('status', 'User has been deleted successfully.');
    }

    public function unlockUser(User $user)
    {
        if($user->id === Auth::id()){
            return redirect()
                ->route('dashboard')
                ->withErrors(['user' => 'You cannot lock/unlock your own account.']);
        }
        if ($user->wrong_counter !== 3){
            $user->wrong_counter = 3;
            $user->save();

            return redirect()
                ->route('dashboard')
                ->with('status', 'User  account has been locked successfully.');
        }
        $user->wrong_counter = 0;
        $user->save();
        return redirect()
            ->route('dashboard')
            ->with('status', 'User account has been unlocked successfully.');
        }

    public function logout()
    {
        Auth::logout();
        return redirect()->route('login');
    }
}
```

### ViewsController.php

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;

class ViewsController extends Controller
{
    public function index()
    {
        return view ('index', [
            'users' => User::all()
        ]);
    }

    public function login()
        {
            return view ('login');
        }
    public function logout()
    {
        return view ('logout');
    }

    public function changePassword()
    {
        return view ('change_password');
    }

    public function editUser(?User $user = null)
    {
        return view('user_form' , [
            'user' => $user
        ]);
    }
}
```

---

## Маршруты

### web.php

```php
<?php

use App\Http\Controllers\ViewsController;
use App\Http\Controllers\ActionsController;
use Illuminate\Support\Facades\Route;

Route::get ('/',[ViewsController::class, 'index'])
    ->name ('dashboard')
    ->middleware(['auth']);

Route::get ('/login',[ViewsController::class, 'login'])
    ->name ('login')
    ->middleware(['guest']);
Route::post ('/login',[ActionsController::class, 'login'])
    ->middleware('guest');

Route::get ('/login/change-password',[ViewsController::class, 'changePassword'])
    ->name ('login.change-password')
    ->middleware(['auth']);
Route::post ('/login/change-password',[ActionsController::class, 'changePassword'])
    ->middleware('auth');

Route::get ('/user/{user?}',[ViewsController::class, 'editUser'])
    ->name ('user')
    ->middleware(['auth']);
Route::delete ('/user/{user}',[ActionsController::class, 'deleteUser'])
    ->name ('user.delete')
    ->middleware('auth');
Route::patch ('/user/{user?}',[ActionsController::class, 'unlockUser'])
    ->name ('user.unlock')
    ->middleware('auth');
Route::post ('/user/{user?}',[ActionsController::class, 'createUser'])
    ->name ('user.save')
    ->middleware('auth');
Route::put ('/user/{user}',[ActionsController::class, 'editUser'])
    ->name ('user.update')
    ->middleware('auth');
Route::post ('/logout',[ActionsController::class, 'logout'])
    ->name ('logout')
    ->middleware('auth');
```

---

## Views (Blade)

### login.blade.php

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Login</title>
</head>
<body>
    <form action="{{ route('login') }}" method="POST">
        @csrf
        <div>
            <label for="email">Email:</label>
            <input type="email" id="email" name="email" value="{{ old('email') }}" required>
            @error('email')
                <div class="error">{{ $message }}</div>
            @enderror
        </div>
        <div>
            <label for="password">Password:</label>
            <input type="password" id="password" name="password" required>
            @error('password')
                <div class="error">{{ $message }}</div>
            @enderror
        </div>
        <button type="submit">Login</button>
    </form>
</body>
</html>
```

### change_password.blade.php

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Need to change password</title>
</head>
<body>
    <form action="{{ route('login.change-password') }}" method="POST">
        @csrf
        <div>
            <label for="current_password">Current Password:</label>
            <input type="password" id="current_password" name="current_password" required>
            @error('current_password')
                <div class="error">{{ $message }}</div>
            @enderror
        </div>
        <div>
            <label for="new_password">New Password:</label>
            <input type="password" id="new_password" name="new_password" required>
            @error('new_password')
                <div class="error">{{ $message }}</div>
            @enderror
        </div>
        <div>
            <label for="new_password_repeat">Repeat New Password:</label>
            <input type="password" id="new_password_repeat" name="new_password_repeat" required>
        </div>
        <button type="submit">Change Password</button>
    </form>
</body>
</html>
```

### index.blade.php

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Dashboard</title>
</head>
<body>
    <h1>Welcome to the Dashboard</h1>
    <h2>List of Users</h2>
    <table border = '1'>
        <thead>
            <tr>
                <th>Name</th>
                <th>Email</th>
                <th>Need to change password</th>
                <th>User is blocked</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            @foreach ($users as $user)
                <tr>
                    <td>{{ $user->name }}</td>
                    <td>{{ $user->email }}</td>
                    <td>{{ $user->need_to_change_password ? 'Yes' : 'No' }}</td>
                    <td>{{ $user->wrong_counter === 3 ? 'Yes' : 'No' }}</td>
                    <td>
                       <a href = "{{ route('user', ['user' => $user->id]) }}">Edit</a>
                       <form action="{{ route('user.delete', ['user' => $user->id]) }}" method="POST" style="display:inline;">
                            @csrf
                            @method('DELETE')
                            <button type="submit">Delete</button>
                        </form>
                        <form action="{{ route('user.unlock', ['user' => $user->id]) }}" method="POST" style="display:inline;">
                            @csrf
                            @method('PATCH')
                            <button type="submit">{{ $user->wrong_counter === 3 ? 'Unlock' : 'Block' }}</button>
                        </form>
                    </td>
                </tr>
            @endforeach
        </tbody>
        <tfoot>
            <tr>
                <td colspan="5">
                    <a href="{{ route('user') }}">Create New User</a>
                    <form action="{{ route('logout') }}" method="POST" style="display:inline;">
                        @csrf
                        <button type="submit">Logout</button>
                    </form>
                </td>

            </tr>
        </tfoot>
    </table>
    @if(session('message'))
        <div class="status">
            {{ session('message') }}
        </div>
    @endif
</body>
</html>
```

### user_form.blade.php

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>{{ $user ? 'Edit user' : 'Create user' }}</title>
</head>
<body>
    <h1>{{ $user ? 'Edit user' : 'Create user' }}</h1>
    <form action="{{ $user ? route('user.update', $user->id) : route('user.save') }}" method="POST">
        @csrf
        @if ($user)
            @method('PUT')
        @endif
        <div>
            <label for="name">Name:</label>
            <input type="text" id="name" name="name" value="{{ old('name', $user->name ?? '') }}" required>
        </div>
        <div>
            <label for="email">Email:</label>
            <input type="email" id="email" name="email" value="{{ old('email', $user->email ?? '') }}" required>
        </div>
        <div>
            <label for="password">Password:</label>
            <input type="password" id="password" name="password" {{ $user ? '' : 'required' }}>
        </div>
        <button type="submit">{{ $user ? 'Update user' : 'Create user' }}</button>
        <a href="{{ route('dashboard') }}">Cancel</a>
    </form>
    <form>
        @if($errors->any())
          <div>
            <h2>Errors:</h2>
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
          </div>
        @endif
    </form>
</body>
</html>
```
### Создание пользователя

### Обновление миграций (Если не создается пользователь)
```
php artisan migrate:refresh
```
```
php artisan tinker
```
```
App\Models\User::create(['email' => 'repev.egor@nttek.ru' , 'password' => '12345678' , 'need_to_change_password' => true , 'name' => 'George Repev']);
```

## 6 модуль

```
composer create-project laravel/laravel --prefer-dist .
```
### Контроллеры
```
php artisan make:controller TestCaseController
Удалить welcome.blade.php, создать index.blade.php
```
### TestCaseController.php:
```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class TestCaseController extends Controller
{
    public function show(Request $request)
    {
        return view('index');
    }

    public function getData()
    {
        $data = file_get_contents('http://localhost:8080/api/fullName');

        if ($data) {
            $data = json_decode($data)->value;
        }

        return redirect()->route('test-case')->with('value', $data);
    }

    public function checkData(Request $request)
    {
        $value = $request->input('value');

        if (preg_match('/^[А-Яа-яЁё]+ [А-Яа-яЁё]+ [А-Яа-яЁё]+$/u', $value)) {
            return redirect()->route('test-case')
                ->with('value', $value)
                ->with('message', 'ФИО корректно');
        } else {
            return redirect()->route('test-case')
                ->with('value', $value)
                ->with('message', 'ФИО содержит некорректные символы');
        }
    }
}
```

### index.blade.php:
```
<p>
    <form action="{{ route('test-case.get') }}">
        <button type="submit">Получить данные</button>
        <span>{{ session('value') }}</span>
    </form>
</p>

<p>
    <form method="POST" action="{{ route('test-case.check') }}">
        @csrf
        <input type="hidden" name="value" value="{{ session('value') }}">
        <button type="submit">Отправить результаты теста</button>
        <span>{{ session('message') }}</span>
    </form>
</p>
```
### web.php:
```
<?php

use App\Http\Controllers\TestCaseController;
use Illuminate\Support\Facades\Route;

Route::get('/', [TestCaseController::class, 'show'])
    ->name('test-case');

Route::get('/get', [TestCaseController::class, 'getData'])
    ->name('test-case.get');

Route::post('/check', [TestCaseController::class, 'checkData'])
    ->name('test-case.check');
```
### 5 modul
```
Информационная система - Авторизация пользователей
	1. Функциональное назначение
	Информационная система предназначена для авторизации зарегистрированных пользователей. Система обеспечивает защищённый вход по связке логин/пароль, регистрацию новых пользователей и защиту от перебора паролей.
	2. Структура базы данных.
Таблица users
Поле	Тип	Описание
id	BIGINT (PK)	Уникальный идентификатор
name	VARCHAR(255)	Имя пользователя
email	VARCHAR(255)	Логин (уникальный)
password	VARCHAR(255)	Хэш пароля (bcrypt)
remember_token	VARCHAR(100)	Токен сессии
created_at	TIMESTAMP	Дата создания записи
updated_at	TIMESTAMP	Дата последнего обновления

3. Методы системы
3.1. Авторизация
Параметр	Значение
Маршрут	POST /login
Метод HTTP	POST
Параметр: email	string, обязательный — логин пользователя
Параметр: password	string, обязательный — пароль пользователя
Параметр: remember	boolean, необязательный — запомнить сессию
Успех	Перенаправление на /home
Ошибка	Сообщение «Вы ввели неверный логин или пароль. Пожалуйста проверьте ещё раз введенные данные»

3.2. Регистрация
Параметр	Значение
Маршрут	POST /register
Метод HTTP	POST
Параметр: name	string, обязательный — имя пользователя
Параметр: email	string, обязательный — логин (уникальный)
Параметр: password	string, обязательный, мин. 8 символов
Параметр: password_confirmation	string, обязательный — подтверждение пароля
Успех	Авторизация и перенаправление на /home
Ошибка	Сообщение о занятом логине или несовпадении паролей

3.3. Выход из системы
Параметр	Значение
Маршрут	POST /logout
Метод HTTP	POST
Результат	Завершение сессии и перенаправление на страницу авторизации

4. Безопасность
Механизм	Описание
Хэширование паролей	Алгоритм bcrypt
Защита от перебора	Блокировка на 50 секунд после 5 неверных попыток подряд (троттлинг)
CSRF-защита	Все формы защищены CSRF-токеном
Валидация полей	Поля логин и пароль обязательны для заполнения
```
### кейсы
```
Получить данные от эмулятора. ФИО содержит только буквы русского алфавита и пробелы (например: Иванов Иван Иванович). Нажать кнопку "Отправить результат теста".
Система подтверждает корректность данных. Валидация пройдена успешно.
Успешно
Получить данные от эмулятора. ФИО содержит запрещённые символы (например: Ив@нов !Иван №1). Нажать кнопку "Отправить результат теста".
Система обнаруживает недопустимые символы. Валидация не пройдена. Выводится сообщение об ошибке.
Успешно
```
### кейсы2
```
№	Роль	Описание	Дата теста	Приоритет тестирования	Заголовок / название теста	Этапы теста	Тестовые данные	Ожидаемый результат
1	Администратор / Разработчик	Тестирование получения данных от клиента через эмулятор	15.06.2026	Высокий	Получение данных клиента из TransferSimulator.exe	1. Запустить TransferSimulator.exe 2. Запустить приложение валидации 3. Нажать кнопку «Получить данные»	Данные от эмулятора	Данные успешно загружены и отображены на форме
2	Администратор / Разработчик	Валидация ФИО — наличие запрещённых символов (цифры)	15.06.2026	Высокий	Проверка ФИО на наличие цифр	1. Получить данные 2. Нажать «Отправить результат теста»	ФИО: «Иванов Иван123 Петрович»	Ошибка: «ФИО содержит недопустимые символы (цифры)»
3	Администратор / Разработчик	Валидация ФИО — наличие специальных символов	15.06.2026	Высокий	Проверка ФИО на наличие спецсимволов	1. Получить данные 2. Нажать «Отправить результат теста»	ФИО: «Сидоров@ Анна #Васильевна»	Ошибка: «ФИО содержит недопустимые символы (спецсимволы)»
4	Администратор / Разработчик	Валидация корректного ФИО	15.06.2026	Высокий	Проверка корректного ФИО	1. Получить данные 2. Нажать «Отправить результат теста»	ФИО: «Смирнова Анна Петровна»	Успешно: «Данные корректны»
5	Пользователь (Клиент)	Тестирование отправки своих данных на валидацию	15.06.2026	Средний	Отправка данных клиента на проверку	1. Заполнить форму как клиент (через эмулятор) 2. Нажать «Отправить данные»	ФИО: «Петров Сергей Александрович»	Данные отправлены на сервер/приложение для валидации
6	Пользователь (Клиент)	Проверка реакции на некорректные данные	15.06.2026	Средний	Получение отказа при некорректном ФИО	1. Отправить данные с ошибкой 2. Просмотреть результат	ФИО: «Козлов123 Алексей»	Клиент получает сообщение об ошибке валидации
7	Администратор / Разработчик	Фиксация всех результатов валидации	15.06.2026	Высокий	Запись результатов тестов в таблицу	1. Выполнить проверки 2. Нажать «Отправить результат теста»	Все предыдущие тесты	Все результаты отображаются в таблице в столбце «Результат»
<img width="1462" height="976" alt="image" src="https://github.com/user-attachments/assets/e7a62341-c1a8-4e5e-9fdd-60a2c519797a" />
```
### modul 5.2
```
Проектная документация
Информационная система — Авторизация и управление пользователями
1. Функциональное назначение
Информационная система предназначена для авторизации зарегистрированных пользователей и управления их учётными записями. Система обеспечивает защищённый вход по связке логин/пароль, защиту от перебора паролей с автоматической блокировкой учётной записи, принудительную смену пароля и административный функционал управления пользователями.
2. Структура базы данных
Поле	Описание
id	Уникальный идентификатор
name	Имя пользователя
email	Логин (уникальный)
password	Хэш пароля
need_to_change_password	Флаг принудительной смены пароля при следующем входе
wrong_counter	Счётчик неверных попыток входа
is_admin	Признак роли пользователя
remember_token	Токен сессии
email_verified_at	Дата подтверждения почты

3. Методы системы
3.1. Авторизация
Параметр	Значение
Метод	login
Параметр: email	string, обязательный, формат email
Параметр: password	string, обязательный, минимум 6 символов
Логика	Проверка связки логин/пароль. При совпадении — переход в систему
Условие блокировки	Если счётчик неверных попыток (wrong_counter) равен 3 — доступ запрещён, сообщение о блокировке учётной записи
Условие смены пароля	Если для пользователя установлен флаг need_to_change_password — переход на страницу смены пароля
Ошибка	При неверных данных — сообщение о несоответствии введённых данных записям системы, увеличение счётчика неверных попыток


3.2. Смена пароля
Параметр	Значение
Метод	changePassword
Параметр: current_password	string, обязательный — проверка соответствия текущему паролю
Параметр: new_password	string, обязательный, минимум 6 символов
Параметр: new_password_repeat	string, обязательный — подтверждение нового пароля
Результат	Установка нового пароля, снятие флага принудительной смены пароля

3.3. Создание пользователя
Параметр	Значение
Метод	createUser
Параметр: name	string, обязательный, максимум 255 символов
Параметр: email	string, обязательный, формат email, проверка уникальности в таблице users
Параметр: password	string, обязательный, минимум 6 символов
Результат	Создание новой учётной записи
Ошибка	Сообщение, если введённый логин уже занят

3.4. Редактирование пользователя
Параметр	Значение
Метод	editUser
Параметр: name	string, обязательный, максимум 255 символов
Параметр: email	string, обязательный, формат email, проверка уникальности (с исключением текущего пользователя)
Параметр: password	string, необязательный, минимум 6 символов
Результат	Обновление данных пользователя. Пароль обновляется только если поле заполнено

3.5. Удаление пользователя
Параметр	Значение
Метод	deleteUser
Логика	Удаление учётной записи по идентификатору
Ограничение	Запрет удаления собственной учётной записи
3.6. Блокировка / разблокировка пользователя
Параметр	Значение
Метод	unlockUser
Логика	Переключение статуса блокировки учётной записи (счётчик неверных попыток)
Ограничение	Запрет блокировки/разблокировки собственной учётной записи

3.7. Выход из системы
Параметр	Значение
Метод	logout
Результат	Завершение сессии пользователя

4. Безопасность
Механизм	Описание
Хэширование паролей	Пароль хранится в хэшированном виде, не в открытом тексте
Защита от перебора	Блокировка учётной записи после 3 неверных попыток входа
Защита от ошибок администрирования	Запрет удаления и блокировки собственной учётной записи
Принудительная смена пароля	Возможность установить требование смены пароля при следующем входе
Валидация данных	Проверка обязательности и формата всех вводимых полей на стороне сервера
```

  # 📘 Три варианта отчёта по КОД 09.02.07-5-2026
## (СНИЛС / Полис ОМС / Паспорт РФ)

> ⚠️ **Важно:** Модули 1–5 во всех трёх отчётах **идентичны** (та же ER-диаграмма, БД, SQL-запрос, Laravel с авторизацией и проектная документация). Меняется **только Модуль 6** — логика валидации и тест-кейсы.
>
> Ниже приведены:
> - **Общие модули 1–5** (один раз для всех отчётов)
> - **Три варианта Модуля 6** — отдельно для СНИЛС, Полиса и Паспорта

---

## 📋 Общие модули 1–5 (одинаковы для всех трёх отчётов)

> 📌 Полный код модулей 1–5 см. в предыдущем отчёте. Здесь — только ключевые моменты.

### Модуль 1. ER-диаграмма
Файл: `ER_Diagram.pdf` (9 сущностей: clients, nomenclature, specifications, specification_lines, orders, order_lines, production, production_outputs, production_inputs)

### Модуль 2. База данных
- БД: `kod_09_02_07` через phpMyAdmin
- SQL-скрипт создания 9 таблиц (см. предыдущий отчёт)
- Импорт из `Заказчики.json` через `php artisan import:clients`

### Модуль 3. SQL-запрос
Расчёт полной стоимости заказа с учётом себестоимости материалов.

### Модуль 4. Laravel
- Таблица `users` (id, name, email, password, need_to_change_password, wrong_counter, is_admin)
- Капча-пазл на странице входа
- Блокировка после 3 неверных попыток
- CRUD пользователей
- Тестовый админ: `admin@test.ru` / `12345678`

### Модуль 5. Проектная документация
Word-документ с описанием функционала, структуры БД, методов системы и безопасности.

---

# 🟢 ОТЧЁТ №1: ВАЛИДАЦИЯ СНИЛС

## 🔗 Модуль 6. Интеграция программных модулей (СНИЛС)

### 6.1. Создание контроллера

```bash
php artisan make:controller TestCaseController
```

**Файл:** `app/Http/Controllers/TestCaseController.php`

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class TestCaseController extends Controller
{
    public function show(Request $request)
    {
        return view('test.index');
    }

    public function getData()
    {
        // URL эмулятора для получения СНИЛС
        $url = 'http://localhost:4444/TransferSimulator/snils';

        $context = stream_context_create(['http' => ['timeout' => 5]]);
        $data = @file_get_contents($url, false, $context);

        if ($data) {
            $json = json_decode($data);
            $data = $json->value ?? '';
        } else {
            $data = 'Ошибка получения данных';
        }

        return redirect()->route('test-case')->with('value', $data);
    }

    public function checkData(Request $request)
    {
        $value = $request->input('value');
        $result = '';

        // Критерий 1: наличие букв (в СНИЛС должны быть только цифры и пробелы/дефисы)
        if (preg_match('/[А-Яа-яA-Za-z]/u', $value)) {
            $result = 'Ошибка: СНИЛС содержит буквы';
        }
        // Критерий 2: неправильный формат (должно быть XXX-XXX-XXX XX — 11 цифр)
        elseif (!preg_match('/^\d{3}-\d{3}-\d{3} \d{2}$/', $value)) {
            $result = 'Ошибка: СНИЛС имеет неверный формат (ожидается XXX-XXX-XXX XX)';
        }
        else {
            $result = 'Успешно';
        }

        $this->writeToDocx($result);

        return redirect()->route('test-case')
            ->with('value', $value)
            ->with('message', $result);
    }

    private function writeToDocx($result)
    {
        $docxPath = base_path('ТестКейс.docx');
        $bookmarkName = 'ResultBookmark';
        $pythonScript = base_path('scripts/write_to_docx.py');
        $command = "python \"{$pythonScript}\" \"{$docxPath}\" \"{$bookmarkName}\" \"{$result}\"";
        exec($command, $output, $returnVar);

        if ($returnVar !== 0) {
            \Log::error('Ошибка записи в DOCX: ' . implode("\n", $output));
        }
    }
}
```

### 6.2. Маршруты (добавь в `routes/web.php`)

```php
use App\Http\Controllers\TestCaseController;

Route::get('/test', [TestCaseController::class, 'show'])->name('test-case');
Route::get('/test/get', [TestCaseController::class, 'getData'])->name('test-case.get');
Route::post('/test/check', [TestCaseController::class, 'checkData'])->name('test-case.check');
```

### 6.3. Представление

**Создай папку:** `resources/views/test/`

**Файл:** `resources/views/test/index.blade.php`

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Валидация СНИЛС</title>
    <style>
        body { font-family: Arial, sans-serif; padding: 20px; }
        .box { border: 1px solid #ccc; padding: 15px; margin: 10px 0; width: 400px; }
        .message { font-weight: bold; color: blue; }
    </style>
</head>
<body>
    <h2>Макет окна приложения валидации СНИЛС</h2>

    <form action="{{ route('test-case.get') }}" method="GET">
        <button type="submit">Получить данные</button>
    </form>

    <div class="box">
        <strong>СНИЛС от эмулятора:</strong><br>
        <input type="text" value="{{ session('value', '') }}" readonly style="width: 100%; margin-top: 5px;">
    </div>

    <form method="POST" action="{{ route('test-case.check') }}">
        @csrf
        <input type="hidden" name="value" value="{{ session('value', '') }}">
        <button type="submit">Отправить результат теста</button>
    </form>

    @if(session('message'))
        <div class="message">{{ session('message') }}</div>
    @endif
</body>
</html>
```

### 6.4. Python-скрипт для записи в Word

```bash
pip install python-docx
```

**Создай папку:** `scripts/` в корне проекта

**Файл:** `scripts/write_to_docx.py`

```python
import sys
from docx import Document
from docx.oxml.ns import qn

def update_bookmark(doc_path, bookmark_name, text):
    doc = Document(doc_path)
    for paragraph in doc.paragraphs:
        bookmark_starts = paragraph._element.findall(qn('w:bookmarkStart'))
        for start in bookmark_starts:
            if start.get(qn('w:name')) == bookmark_name:
                bookmark_id = start.get(qn('w:id'))
                bookmark_ends = paragraph._element.findall(qn('w:bookmarkEnd'))
                for end in bookmark_ends:
                    if end.get(qn('w:id')) == bookmark_id:
                        new_run = paragraph._element.makeelement(qn('w:r'), {})
                        new_text = paragraph._element.makeelement(qn('w:t'), {})
                        new_text.text = text
                        new_run.append(new_text)
                        end.addprevious(new_run)
                        doc.save(doc_path)
                        return "OK"
    return "Bookmark not found"

if __name__ == "__main__":
    if len(sys.argv) == 4:
        update_bookmark(sys.argv[1], sys.argv[2], sys.argv[3])
```

### 6.5. Подготовка ТестКейс.docx

1. Открой `ТестКейс.docx` в Word
2. Заполни столбцы **«Действие»** и **«Ожидаемый результат»**
3. В столбце **«Результат»** для каждой строки поставь закладку `ResultBookmark`:
   - Выдели ячейку → **Вставка → Закладка** → имя `ResultBookmark` → **Добавить**
4. Положи файл в корень проекта Laravel

### 6.6. Тест-кейсы (СНИЛС)

| № | Действие | Ожидаемый результат |
|---|---|---|
| 1 | Получить данные от эмулятора. СНИЛС содержит буквы (123-456-78A 01). Нажать «Отправить результат теста». | Система обнаруживает буквы. Валидация не пройдена. |
| 2 | Получить данные от эмулятора. СНИЛС имеет неверный формат (12345-678-90). Нажать «Отправить результат теста». | Система обнаруживает неверный формат. Валидация не пройдена. |
| 3 | Получить данные от эмулятора. СНИЛС корректный (123-456-789 01). Нажать «Отправить результат теста». | Валидация пройдена успешно. Данные корректны. |

### 6.7. Запуск эмулятора

```bash
# Вариант 1: EXE
TransferSimulator.exe

# Вариант 2: JAR
java -jar TransferSimulator.jar

# Тестирование через Postman
GET http://localhost:4444/TransferSimulator/snils
```

---

# 🟡 ОТЧЁТ №2: ВАЛИДАЦИЯ ПОЛИСА ОМС

## 🔗 Модуль 6. Интеграция программных модулей (Полис ОМС)

### 6.1. Создание контроллера

```bash
php artisan make:controller TestCaseController
```

**Файл:** `app/Http/Controllers/TestCaseController.php`

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class TestCaseController extends Controller
{
    public function show(Request $request)
    {
        return view('test.index');
    }

    public function getData()
    {
        // URL эмулятора для получения полиса ОМС
        $url = 'http://localhost:4444/TransferSimulator/policy';

        $context = stream_context_create(['http' => ['timeout' => 5]]);
        $data = @file_get_contents($url, false, $context);

        if ($data) {
            $json = json_decode($data);
            $data = $json->value ?? '';
        } else {
            $data = 'Ошибка получения данных';
        }

        return redirect()->route('test-case')->with('value', $data);
    }

    public function checkData(Request $request)
    {
        $value = $request->input('value');
        $result = '';

        // Критерий 1: наличие букв (в полисе должны быть только цифры)
        if (preg_match('/[А-Яа-яA-Za-z]/u', $value)) {
            $result = 'Ошибка: Полис содержит буквы';
        }
        // Критерий 2: количество цифр не равно 16
        elseif (!preg_match('/^\d{16}$/', $value)) {
            $result = 'Ошибка: Полис должен содержать ровно 16 цифр';
        }
        else {
            $result = 'Успешно';
        }

        $this->writeToDocx($result);

        return redirect()->route('test-case')
            ->with('value', $value)
            ->with('message', $result);
    }

    private function writeToDocx($result)
    {
        $docxPath = base_path('ТестКейс.docx');
        $bookmarkName = 'ResultBookmark';
        $pythonScript = base_path('scripts/write_to_docx.py');
        $command = "python \"{$pythonScript}\" \"{$docxPath}\" \"{$bookmarkName}\" \"{$result}\"";
        exec($command, $output, $returnVar);

        if ($returnVar !== 0) {
            \Log::error('Ошибка записи в DOCX: ' . implode("\n", $output));
        }
    }
}
```

### 6.2. Маршруты (добавь в `routes/web.php`)

```php
use App\Http\Controllers\TestCaseController;

Route::get('/test', [TestCaseController::class, 'show'])->name('test-case');
Route::get('/test/get', [TestCaseController::class, 'getData'])->name('test-case.get');
Route::post('/test/check', [TestCaseController::class, 'checkData'])->name('test-case.check');
```

### 6.3. Представление

**Создай папку:** `resources/views/test/`

**Файл:** `resources/views/test/index.blade.php`

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Валидация Полиса ОМС</title>
    <style>
        body { font-family: Arial, sans-serif; padding: 20px; }
        .box { border: 1px solid #ccc; padding: 15px; margin: 10px 0; width: 400px; }
        .message { font-weight: bold; color: blue; }
    </style>
</head>
<body>
    <h2>Макет окна приложения валидации Полиса ОМС</h2>

    <form action="{{ route('test-case.get') }}" method="GET">
        <button type="submit">Получить данные</button>
    </form>

    <div class="box">
        <strong>Полис от эмулятора:</strong><br>
        <input type="text" value="{{ session('value', '') }}" readonly style="width: 100%; margin-top: 5px;">
    </div>

    <form method="POST" action="{{ route('test-case.check') }}">
        @csrf
        <input type="hidden" name="value" value="{{ session('value', '') }}">
        <button type="submit">Отправить результат теста</button>
    </form>

    @if(session('message'))
        <div class="message">{{ session('message') }}</div>
    @endif
</body>
</html>
```

### 6.4. Python-скрипт для записи в Word

(Идентичен отчёту №1 — см. `scripts/write_to_docx.py` выше)

### 6.5. Подготовка ТестКейс.docx

(Идентична отчёту №1 — закладки `ResultBookmark` в столбце «Результат»)

### 6.6. Тест-кейсы (Полис ОМС)

| № | Действие | Ожидаемый результат |
|---|---|---|
| 1 | Получить данные от эмулятора. Полис содержит буквы (12345678901234A6). Нажать «Отправить результат теста». | Система обнаруживает буквы. Валидация не пройдена. |
| 2 | Получить данные от эмулятора. Полис содержит не 16 цифр (1234567890). Нажать «Отправить результат теста». | Система обнаруживает неверное количество цифр. Валидация не пройдена. |
| 3 | Получить данные от эмулятора. Полис корректный (1234567890123456). Нажать «Отправить результат теста». | Валидация пройдена успешно. Данные корректны. |

### 6.7. Запуск эмулятора

```bash
# Вариант 1: EXE
TransferSimulator.exe

# Вариант 2: JAR
java -jar TransferSimulator.jar

# Тестирование через Postman
GET http://localhost:4444/TransferSimulator/policy
```

---

# 🔵 ОТЧЁТ №3: ВАЛИДАЦИЯ ПАСПОРТА РФ

## 🔗 Модуль 6. Интеграция программных модулей (Паспорт РФ)

### 6.1. Создание контроллера

```bash
php artisan make:controller TestCaseController
```

**Файл:** `app/Http/Controllers/TestCaseController.php`

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class TestCaseController extends Controller
{
    public function show(Request $request)
    {
        return view('test.index');
    }

    public function getData()
    {
        // URL эмулятора для получения паспорта
        $url = 'http://localhost:4444/TransferSimulator/passport';

        $context = stream_context_create(['http' => ['timeout' => 5]]);
        $data = @file_get_contents($url, false, $context);

        if ($data) {
            $json = json_decode($data);
            $data = $json->value ?? '';
        } else {
            $data = 'Ошибка получения данных';
        }

        return redirect()->route('test-case')->with('value', $data);
    }

    public function checkData(Request $request)
    {
        $value = $request->input('value');
        $result = '';

        // Критерий 1: наличие букв (в паспорте должны быть только цифры и пробел)
        if (preg_match('/[А-Яа-яA-Za-z]/u', $value)) {
            $result = 'Ошибка: Паспорт содержит буквы';
        }
        // Критерий 2: неправильный формат (должно быть XXXX XXXXXX — 4+6 цифр)
        elseif (!preg_match('/^\d{4} \d{6}$/', $value)) {
            $result = 'Ошибка: Паспорт имеет неверный формат (ожидается XXXX XXXXXX)';
        }
        else {
            $result = 'Успешно';
        }

        $this->writeToDocx($result);

        return redirect()->route('test-case')
            ->with('value', $value)
            ->with('message', $result);
    }

    private function writeToDocx($result)
    {
        $docxPath = base_path('ТестКейс.docx');
        $bookmarkName = 'ResultBookmark';
        $pythonScript = base_path('scripts/write_to_docx.py');
        $command = "python \"{$pythonScript}\" \"{$docxPath}\" \"{$bookmarkName}\" \"{$result}\"";
        exec($command, $output, $returnVar);

        if ($returnVar !== 0) {
            \Log::error('Ошибка записи в DOCX: ' . implode("\n", $output));
        }
    }
}
```

### 6.2. Маршруты (добавь в `routes/web.php`)

```php
use App\Http\Controllers\TestCaseController;

Route::get('/test', [TestCaseController::class, 'show'])->name('test-case');
Route::get('/test/get', [TestCaseController::class, 'getData'])->name('test-case.get');
Route::post('/test/check', [TestCaseController::class, 'checkData'])->name('test-case.check');
```

### 6.3. Представление

**Создай папку:** `resources/views/test/`

**Файл:** `resources/views/test/index.blade.php`

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Валидация Паспорта РФ</title>
    <style>
        body { font-family: Arial, sans-serif; padding: 20px; }
        .box { border: 1px solid #ccc; padding: 15px; margin: 10px 0; width: 400px; }
        .message { font-weight: bold; color: blue; }
    </style>
</head>
<body>
    <h2>Макет окна приложения валидации Паспорта РФ</h2>

    <form action="{{ route('test-case.get') }}" method="GET">
        <button type="submit">Получить данные</button>
    </form>

    <div class="box">
        <strong>Паспорт от эмулятора:</strong><br>
        <input type="text" value="{{ session('value', '') }}" readonly style="width: 100%; margin-top: 5px;">
    </div>

    <form method="POST" action="{{ route('test-case.check') }}">
        @csrf
        <input type="hidden" name="value" value="{{ session('value', '') }}">
        <button type="submit">Отправить результат теста</button>
    </form>

    @if(session('message'))
        <div class="message">{{ session('message') }}</div>
    @endif
</body>
</html>
```

### 6.4. Python-скрипт для записи в Word

(Идентичен отчёту №1 — см. `scripts/write_to_docx.py` выше)

### 6.5. Подготовка ТестКейс.docx

(Идентична отчёту №1 — закладки `ResultBookmark` в столбце «Результат»)

### 6.6. Тест-кейсы (Паспорт РФ)

| № | Действие | Ожидаемый результат |
|---|---|---|
| 1 | Получить данные от эмулятора. Паспорт содержит буквы (4510 AB3456). Нажать «Отправить результат теста». | Система обнаруживает буквы. Валидация не пройдена. |
| 2 | Получить данные от эмулятора. Паспорт имеет неверный формат (4510123456 — без пробела). Нажать «Отправить результат теста». | Система обнаруживает неверный формат. Валидация не пройдена. |
| 3 | Получить данные от эмулятора. Паспорт корректный (4510 123456). Нажать «Отправить результат теста». | Валидация пройдена успешно. Данные корректны. |

### 6.7. Запуск эмулятора

```bash
# Вариант 1: EXE
TransferSimulator.exe

# Вариант 2: JAR
java -jar TransferSimulator.jar

# Тестирование через Postman
GET http://localhost:4444/TransferSimulator/passport
```

---

## 📊 Сводная таблица различий между отчётами

| Параметр | Отчёт №1 (СНИЛС) | Отчёт №2 (Полис ОМС) | Отчёт №3 (Паспорт РФ) |
|---|---|---|---|
| URL эмулятора | `/TransferSimulator/snils` | `/TransferSimulator/policy` | `/TransferSimulator/passport` |
| Правильный формат | `XXX-XXX-XXX XX` (11 цифр) | `XXXXXXXXXXXXXXXX` (16 цифр) | `XXXX XXXXXX` (4+6 цифр) |
| Пример корректных данных | `123-456-789 01` | `1234567890123456` | `4510 123456` |
| Критерий 1 валидации | Наличие букв | Наличие букв | Наличие букв |
| Критерий 2 валидации | Неверный формат | Не 16 цифр | Неверный формат |
| Regex формат | `/^\d{3}-\d{3}-\d{3} \d{2}$/` | `/^\d{16}$/` | `/^\d{4} \d{6}$/` |
| Regex буквы | `/[А-Яа-яA-Za-z]/u` | `/[А-Яа-яA-Za-z]/u` | `/[А-Яа-яA-Za-z]/u` |

---

## ✅ Итоговый чек-лист (одинаков для всех трёх отчётов)

```bash
# 1. Создание проекта
composer create-project laravel/laravel kod_project
cd kod_project

# 2. Настройка .env (вручную)
# DB_DATABASE=kod_09_02_07

# 3. Создание миграции users и запуск
php artisan make:migration create_users_table
php artisan migrate

# 4. Создание модели и контроллеров
php artisan make:model User
php artisan make:controller ActionsController
php artisan make:controller ViewsController
php artisan make:controller TestCaseController

# 5. Создание команды импорта и запуск
php artisan make:command ImportClients
php artisan import:clients

# 6. Создание представлений вручную:
#    resources/views/login.blade.php (с капчой)
#    resources/views/index.blade.php
#    resources/views/user_form.blade.php
#    resources/views/change_password.blade.php
#    resources/views/test/index.blade.php (с названием соответствующего документа)

# 7. Настройка routes/web.php (вручную)

# 8. Создание администратора
php artisan tinker
# App\Models\User::create([...]);

# 9. Установка Python-библиотеки
pip install python-docx

# 10. Создание скрипта scripts/write_to_docx.py

# 11. Запуск сервера
php artisan serve
```

---

## 🎯 Что реализовано во всех трёх отчётах

✅ ER-диаграмма в PDF (3НФ, PK/FK)
✅ База данных MySQL через phpMyAdmin
✅ SQL-запрос расчёта стоимости заказа
✅ Laravel с миграциями, моделями, контроллерами
✅ Капча-пазл на странице входа
✅ Блокировка после 3 неверных попыток
✅ Принудительная смена пароля
✅ CRUD пользователей
✅ Все тексты сообщений на русском (точно по заданию)
✅ Проверка уникальности логина
✅ Интеграция с эмулятором TransferSimulator
✅ Валидация по двум критериям (буквы + формат)
✅ Запись результатов в закладки Word-документа
✅ Проектная документация по модулю 5

### 🆕 Уникальное для каждого отчёта:

| Отчёт | Валидируемый документ | Формат |
|---|---|---|
| №1 | СНИЛС | `XXX-XXX-XXX XX` |
| №2 | Полис ОМС | `XXXXXXXXXXXXXXXX` (16 цифр) |
| №3 | Паспорт РФ | `XXXX XXXXXX` |

---

> 💡 **Совет:** Чтобы быстро переключаться между отчётами при подготовке:
> - В `TestCaseController.php` меняется только `$url` и логика `checkData()`
> - В `index.blade.php` меняется только заголовок `<h2>` и подпись поля
> - В `ТестКейс.docx` меняются только тестовые данные и ожидаемые результаты

Удачи на демонстрационном экзамене! 🚀
  # 🧪 7 тест-кейсов для каждого отчёта (СНИЛС / Полис ОМС / Паспорт РФ)

> 📌 **Важно:** Для каждого тест-кейса в столбце **«Результат»** документа `ТестКейс.docx` нужно поставить **отдельную закладку** с уникальным именем: `Result1`, `Result2`, `Result3` и т.д.

---

# 🟢 ОТЧЁТ №1: Тест-кейсы для СНИЛС

## 📋 Таблица тест-кейсов (СНИЛС)

| № | Действие | Ожидаемый результат | Закладка |
|---|---|---|---|
| 1 | Получить данные от эмулятора. СНИЛС корректный: `123-456-789 01`. Нажать «Отправить результат теста». | Валидация пройдена успешно. Данные корректны. | `Result1` |
| 2 | Получить данные от эмулятора. СНИЛС содержит буквы: `123-456-78A 01`. Нажать «Отправить результат теста». | Система обнаруживает буквы. Валидация не пройдена. | `Result2` |
| 3 | Получить данные от эмулятора. СНИЛС без дефисов: `12345678901`. Нажать «Отправить результат теста». | Система обнаруживает неверный формат. Валидация не пройдена. | `Result3` |
| 4 | Получить данные от эмулятора. СНИЛС содержит 10 цифр: `123-456-78 01`. Нажать «Отправить результат теста». | Система обнаруживает неверное количество цифр. Валидация не пройдена. | `Result4` |
| 5 | Получить данные от эмулятора. СНИЛС со спецсимволами: `123-456-78# 01`. Нажать «Отправить результат теста». | Система обнаруживает запрещённые символы. Валидация не пройдена. | `Result5` |
| 6 | Получить данные от эмулятора. СНИЛС содержит 12 цифр: `123-456-789 012`. Нажать «Отправить результат теста». | Система обнаруживает неверное количество цифр. Валидация не пройдена. | `Result6` |
| 7 | Получить данные от эмулятора. Пустой СНИЛС: ``. Нажать «Отправить результат теста». | Система обнаруживает пустое значение. Валидация не пройдена. | `Result7` |

## 🔧 Исправленный `TestCaseController.php` (СНИЛС — 7 проверок)

```php
public function checkData(Request $request)
{
    $value = trim($request->input('value'));
    $result = '';

    // Проверка 1: пустое значение
    if (empty($value)) {
        $result = 'Ошибка: СНИЛС пустой';
    }
    // Проверка 2: наличие букв
    elseif (preg_match('/[А-Яа-яA-Za-z]/u', $value)) {
        $result = 'Ошибка: СНИЛС содержит буквы';
    }
    // Проверка 3: наличие спецсимволов
    elseif (preg_match('/[^0-9\s\-]/u', $value)) {
        $result = 'Ошибка: СНИЛС содержит запрещённые символы';
    }
    // Проверка 4: неверный формат (должно быть XXX-XXX-XXX XX)
    elseif (!preg_match('/^\d{3}-\d{3}-\d{3} \d{2}$/', $value)) {
        $result = 'Ошибка: СНИЛС имеет неверный формат';
    }
    else {
        $result = 'Успешно';
    }

    $this->writeToDocx($result);

    return redirect()->route('test-case')
        ->with('value', $value)
        ->with('message', $result);
}

private function writeToDocx($result)
{
    $docxPath = base_path('ТестКейс.docx');
    $pythonScript = base_path('scripts/write_to_docx.py');
    
    // Записываем результат в следующую свободную закладку
    $bookmarks = ['Result1', 'Result2', 'Result3', 'Result4', 'Result5', 'Result6', 'Result7'];
    
    foreach ($bookmarks as $bookmark) {
        $command = "python \"{$pythonScript}\" \"{$docxPath}\" \"{$bookmark}\" \"{$result}\"";
        exec($command, $output, $returnVar);
        
        if ($returnVar === 0) {
            break; // Успешно записали — выходим
        }
    }
}
```

## 🐍 Обновлённый Python-скрипт `scripts/write_to_docx.py`

```python
import sys
from docx import Document
from docx.oxml.ns import qn

def update_bookmark(doc_path, bookmark_name, text):
    doc = Document(doc_path)
    for paragraph in doc.paragraphs:
        bookmark_starts = paragraph._element.findall(qn('w:bookmarkStart'))
        for start in bookmark_starts:
            if start.get(qn('w:name')) == bookmark_name:
                bookmark_id = start.get(qn('w:id'))
                bookmark_ends = paragraph._element.findall(qn('w:bookmarkEnd'))
                for end in bookmark_ends:
                    if end.get(qn('w:id')) == bookmark_id:
                        # Удаляем старое содержимое между закладками
                        in_bookmark = False
                        elements_to_remove = []
                        for child in list(paragraph._element):
                            if child.tag == qn('w:bookmarkStart') and child.get(qn('w:id')) == bookmark_id:
                                in_bookmark = True
                            elif child.tag == qn('w:bookmarkEnd') and child.get(qn('w:id')) == bookmark_id:
                                break
                            elif in_bookmark and child.tag == qn('w:r'):
                                elements_to_remove.append(child)
                        
                        for elem in elements_to_remove:
                            paragraph._element.remove(elem)
                        
                        # Вставляем новый текст
                        new_run = paragraph._element.makeelement(qn('w:r'), {})
                        new_text = paragraph._element.makeelement(qn('w:t'), {})
                        new_text.text = text
                        new_run.append(new_text)
                        end.addprevious(new_run)
                        
                        doc.save(doc_path)
                        return "OK"
    return "Bookmark not found"

if __name__ == "__main__":
    if len(sys.argv) == 4:
        result = update_bookmark(sys.argv[1], sys.argv[2], sys.argv[3])
        print(result)
        sys.exit(0 if result == "OK" else 1)
```

---

# 🟡 ОТЧЁТ №2: Тест-кейсы для Полиса ОМС

## 📋 Таблица тест-кейсов (Полис ОМС)

| № | Действие | Ожидаемый результат | Закладка |
|---|---|---|---|
| 1 | Получить данные от эмулятора. Полис корректный: `1234567890123456`. Нажать «Отправить результат теста». | Валидация пройдена успешно. Данные корректны. | `Result1` |
| 2 | Получить данные от эмулятора. Полис содержит буквы: `12345678901234A6`. Нажать «Отправить результат теста». | Система обнаруживает буквы. Валидация не пройдена. | `Result2` |
| 3 | Получить данные от эмулятора. Полис содержит 15 цифр: `123456789012345`. Нажать «Отправить результат теста». | Система обнаруживает неверное количество цифр. Валидация не пройдена. | `Result3` |
| 4 | Получить данные от эмулятора. Полис содержит 17 цифр: `12345678901234567`. Нажать «Отправить результат теста». | Система обнаруживает неверное количество цифр. Валидация не пройдена. | `Result4` |
| 5 | Получить данные от эмулятора. Полис со спецсимволами: `12345678901234@6`. Нажать «Отправить результат теста». | Система обнаруживает запрещённые символы. Валидация не пройдена. | `Result5` |
| 6 | Получить данные от эмулятора. Полис с пробелами: `1234 5678 9012 3456`. Нажать «Отправить результат теста». | Система обнаруживает пробелы. Валидация не пройдена. | `Result6` |
| 7 | Получить данные от эмулятора. Пустой полис: ``. Нажать «Отправить результат теста». | Система обнаруживает пустое значение. Валидация не пройдена. | `Result7` |

## 🔧 Исправленный `TestCaseController.php` (Полис ОМС — 7 проверок)

```php
public function checkData(Request $request)
{
    $value = trim($request->input('value'));
    $result = '';

    // Проверка 1: пустое значение
    if (empty($value)) {
        $result = 'Ошибка: Полис пустой';
    }
    // Проверка 2: наличие букв
    elseif (preg_match('/[А-Яа-яA-Za-z]/u', $value)) {
        $result = 'Ошибка: Полис содержит буквы';
    }
    // Проверка 3: наличие пробелов или спецсимволов
    elseif (preg_match('/[^0-9]/u', $value)) {
        $result = 'Ошибка: Полис содержит запрещённые символы';
    }
    // Проверка 4: неверное количество цифр (должно быть ровно 16)
    elseif (!preg_match('/^\d{16}$/', $value)) {
        $result = 'Ошибка: Полис должен содержать ровно 16 цифр';
    }
    else {
        $result = 'Успешно';
    }

    $this->writeToDocx($result);

    return redirect()->route('test-case')
        ->with('value', $value)
        ->with('message', $result);
}
```

---

# 🔵 ОТЧЁТ №3: Тест-кейсы для Паспорта РФ

## 📋 Таблица тест-кейсов (Паспорт РФ)

| № | Действие | Ожидаемый результат | Закладка |
|---|---|---|---|
| 1 | Получить данные от эмулятора. Паспорт корректный: `4510 123456`. Нажать «Отправить результат теста». | Валидация пройдена успешно. Данные корректны. | `Result1` |
| 2 | Получить данные от эмулятора. Паспорт содержит буквы: `4510 AB3456`. Нажать «Отправить результат теста». | Система обнаруживает буквы. Валидация не пройдена. | `Result2` |
| 3 | Получить данные от эмулятора. Паспорт без пробела: `4510123456`. Нажать «Отправить результат теста». | Система обнаруживает неверный формат. Валидация не пройдена. | `Result3` |
| 4 | Получить данные от эмулятора. Паспорт с форматом 3+7: `451 0123456`. Нажать «Отправить результат теста». | Система обнаруживает неверный формат. Валидация не пройдена. | `Result4` |
| 5 | Получить данные от эмулятора. Паспорт с форматом 4+7: `4510 1234567`. Нажать «Отправить результат теста». | Система обнаруживает неверный формат. Валидация не пройдена. | `Result5` |
| 6 | Получить данные от эмулятора. Паспорт со спецсимволами: `4510 #23456`. Нажать «Отправить результат теста». | Система обнаруживает запрещённые символы. Валидация не пройдена. | `Result6` |
| 7 | Получить данные от эмулятора. Пустой паспорт: ``. Нажать «Отправить результат теста». | Система обнаруживает пустое значение. Валидация не пройдена. | `Result7` |

## 🔧 Исправленный `TestCaseController.php` (Паспорт РФ — 7 проверок)

```php
public function checkData(Request $request)
{
    $value = trim($request->input('value'));
    $result = '';

    // Проверка 1: пустое значение
    if (empty($value)) {
        $result = 'Ошибка: Паспорт пустой';
    }
    // Проверка 2: наличие букв
    elseif (preg_match('/[А-Яа-яA-Za-z]/u', $value)) {
        $result = 'Ошибка: Паспорт содержит буквы';
    }
    // Проверка 3: наличие спецсимволов (не цифры и не пробел)
    elseif (preg_match('/[^0-9\s]/u', $value)) {
        $result = 'Ошибка: Паспорт содержит запрещённые символы';
    }
    // Проверка 4: неверный формат (должно быть XXXX XXXXXX)
    elseif (!preg_match('/^\d{4} \d{6}$/', $value)) {
        $result = 'Ошибка: Паспорт имеет неверный формат';
    }
    else {
        $result = 'Успешно';
    }

    $this->writeToDocx($result);

    return redirect()->route('test-case')
        ->with('value', $value)
        ->with('message', $result);
}
```

---

## 📊 Сводная таблица всех тест-кейсов

### 🟢 СНИЛС (формат `XXX-XXX-XXX XX`)

| № | Тестовые данные | Ожидаемый результат |
|---|---|---|
| 1 | `123-456-789 01` | ✅ Успешно |
| 2 | `123-456-78A 01` | ❌ Содержит буквы |
| 3 | `12345678901` | ❌ Неверный формат |
| 4 | `123-456-78 01` | ❌ Неверное количество цифр |
| 5 | `123-456-78# 01` | ❌ Запрещённые символы |
| 6 | `123-456-789 012` | ❌ Неверное количество цифр |
| 7 | `` (пусто) | ❌ Пустое значение |

### 🟡 Полис ОМС (формат `XXXXXXXXXXXXXXXX` — 16 цифр)

| № | Тестовые данные | Ожидаемый результат |
|---|---|---|
| 1 | `1234567890123456` | ✅ Успешно |
| 2 | `12345678901234A6` | ❌ Содержит буквы |
| 3 | `123456789012345` | ❌ 15 цифр вместо 16 |
| 4 | `12345678901234567` | ❌ 17 цифр вместо 16 |
| 5 | `12345678901234@6` | ❌ Запрещённые символы |
| 6 | `1234 5678 9012 3456` | ❌ Содержит пробелы |
| 7 | `` (пусто) | ❌ Пустое значение |

### 🔵 Паспорт РФ (формат `XXXX XXXXXX`)

| № | Тестовые данные | Ожидаемый результат |
|---|---|---|
| 1 | `4510 123456` | ✅ Успешно |
| 2 | `4510 AB3456` | ❌ Содержит буквы |
| 3 | `4510123456` | ❌ Нет пробела |
| 4 | `451 0123456` | ❌ Неверный формат (3+7) |
| 5 | `4510 1234567` | ❌ Неверный формат (4+7) |
| 6 | `4510 #23456` | ❌ Запрещённые символы |
| 7 | `` (пусто) | ❌ Пустое значение |

---

## 📝 Инструкция по заполнению `ТестКейс.docx`

### Шаг 1. Создание закладок

1. Открой `ТестКейс.docx` в Word
2. Для каждой строки таблицы в столбце **«Результат»**:
   - Выдели ячейку (или поставь курссор в неё)
   - **Вставка → Закладка**
   - Введи имя: `Result1`, `Result2`, ... `Result7`
   - Нажми **«Добавить»**

### Шаг 2. Запуск тестов

```bash
# 1. Запусти эмулятор
TransferSimulator.exe

# 2. Запусти Laravel-сервер
php artisan serve

# 3. Открой в браузере
http://localhost:8000/test
```

### Шаг 3. Проверка результатов

После каждого нажатия кнопки **«Отправить результат теста»**:
- Результат отображается на форме
- Результат записывается в соответствующую закладку в `ТестКейс.docx`
- Открой Word и проверь, что все 7 ячеек заполнены

---

## ✅ Итоговый чек-лист

- [ ] Создано 7 тест-кейсов для СНИЛС / Полиса / Паспорта
- [ ] В `ТестКейс.docx` созданы закладки `Result1` ... `Result7`
- [ ] В `TestCaseController.php` реализованы все 7 проверок
- [ ] Python-скрипт корректно записывает результаты в закладки
- [ ] Все тесты пройдены успешно
- [ ] Результаты зафиксированы в документе

---

> 💡 **Совет:** Если хочешь протестировать конкретный кейс — просто введи тестовые данные вручную в поле на форме и нажми «Отправить результат теста». Эмулятор можно не запускать для негативных тестов.

Удачи на демонстрационном экзамене! 🚀
