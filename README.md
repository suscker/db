# Функциональные требования к системе управления библиотекой
Работу выполнил студент группы 353502 Любашенко А. С.

## Общие требования
Система предназначена для автоматизации процессов выдачи, учета и управления книжным фондом цифровой или физической библиотеки.

## Обязательные функциональные требования
### Авторизация и аутентификация пользователя
- **Регистрация** нового читателя.
- **Вход** в систему по email и паролю.
- **Выход** из системы.
- **Шифрование паролей** с использованием алгоритма bcrypt.

### Управление пользователями (CRUD)
- **Create:** Библиотекарь или администратор может добавлять новых читателей и сотрудников.
- **Read:** Просмотр списка пользователей, фильтрация по роли, поиск по ФИО.
- **Update:** Редактирование профиля пользователя, блокировка учетной записи читателя (например, за просрочку).
- **Delete:** Мягкое удаление (деактивация) учетных записей.

### Система ролей
Реализована ролевая модель доступа (RBAC):
- **Читатель (User):** Может просматривать каталог книг, брать книги на абонемент, продлевать сроки, оставлять отзывы.
- **Библиотекарь (Librarian):** Может управлять экземплярами книг, обрабатывать выдачу и возврат, управлять бронированиями.
- **Администратор (Admin):** Полный доступ ко всем функциям, включая управление пользователями, каталогом и системными настройками.

### Журналирование действий пользователя

### Бизнес-требования
- **Управление каталогом книг:** Добавление, редактирование, удаление книг и их экземпляров.
- **Управление авторами и жанрами:** Ведение справочников.
- **Процесс выдачи и возврата:** Фиксация аренды книги читателем, расчет штрафов за просрочку.
- **Система бронирования:** Читатель может забронировать книгу, если все экземпляры выданы.
- **Поиск и фильтрация:** Удобный поиск по каталогу с фильтрами по жанру, автору, году издания.
- **Рейтинги и отзывы:** Читатели могут оставлять отзывы и оценки книгам.

# Перечень сущностей БД и их обоснование

- users (Пользователи): Основная сущность для читателей и сотрудников библиотеки.
- roles (Роли): Хранит роли (Admin, Librarian, User). Основа системы прав.
- books (Книги): Информация о книге как о издании (название, описание, год). Это "каталог".
- book_items (Экземпляры книг): Конкретные физические/цифровые копии книги. У каждой свой уникальный инвентарный номер. Связь "One-to-Many" с books (одно издание -> много экземпляров).
- authors (Авторы): Справочник авторов.
- genres (Жанры): Справочник жанров.
- publishers (Издательства): Справочник издательств.
- rentals (Выдачи): Самая важная бизнес-сущность. Фиксирует факт выдачи экземпляра читателю.
- reservations (Бронирования): Фиксирует бронь книги читателем.
- reviews (Отзывы): Хранит отзывы и оценки, которые читатели оставляют к книгам (изданиям).
- user_action_logs (Журнал действий): Для аудита.
- book_author (Связь книга-автор): Промежуточная таблица для связи Many-to-Many между books и authors (книга может иметь несколько авторов, автор может иметь много книг).
- book_genre (Связь книга-жанр): Промежуточная таблица для связи Many-to-Many между books и genres (книга может относиться к нескольким жанрам).

## Описание сущностей базы данных

### Таблица `users`
| Имя поля | Тип | Ограничения | Связи |
|----------|-----|-------------|-------|
| id | SERIAL | PRIMARY KEY | |
| email | VARCHAR(255) | UNIQUE, NOT NULL | |
| password_hash | VARCHAR(255) | NOT NULL | |
| first_name | VARCHAR(100) | NOT NULL | |
| last_name | VARCHAR(100) | NOT NULL | |
| role_id | INTEGER | NOT NULL | FOREIGN KEY REFERENCES roles(id) |
| is_active | BOOLEAN | DEFAULT TRUE | |
| created_at | TIMESTAMP | DEFAULT NOW() | |

### Таблица `roles`
| Имя поля | Тип | Ограничения | Связи |
|----------|-----|-------------|-------|
| id | SERIAL | PRIMARY KEY | |
| name | VARCHAR(50) | UNIQUE, NOT NULL | |

### Таблица `books`
| Имя поля | Тип | Ограничения | Связи |
|----------|-----|-------------|-------|
| id | SERIAL | PRIMARY KEY | |
| title | VARCHAR(255) | NOT NULL | |
| isbn | VARCHAR(20) | UNIQUE | |
| description | TEXT | | |
| published_year | INTEGER | | |
| publisher_id | INTEGER | | FOREIGN KEY REFERENCES publishers(id) |
| cover_image_url | VARCHAR(255) | | |

### Таблица `book_items`
| Имя поля | Тип | Ограничения | Связи |
|----------|-----|-------------|-------|
| id | SERIAL | PRIMARY KEY | |
| book_id | INTEGER | NOT NULL | FOREIGN KEY REFERENCES books(id) ON DELETE CASCADE |
| inventory_number | VARCHAR(50) | UNIQUE, NOT NULL | |
| status | book_item_status | NOT NULL DEFAULT 'Available' | |

### Таблица `authors`
| Имя поля | Тип | Ограничения | Связи |
|----------|-----|-------------|-------|
| id | SERIAL | PRIMARY KEY | |
| full_name | VARCHAR(255) | NOT NULL | |

### Таблица `genres`
| Имя поля | Тип | Ограничения | Связи |
|----------|-----|-------------|-------|
| id | SERIAL | PRIMARY KEY | |
| name | VARCHAR(100) | UNIQUE, NOT NULL | |

### Таблица `publishers`
| Имя поля | Тип | Ограничения | Связи |
|----------|-----|-------------|-------|
| id | SERIAL | PRIMARY KEY | |
| name | VARCHAR(255) | UNIQUE, NOT NULL | |

### Таблица `rentals`
| Имя поля | Тип | Ограничения | Связи |
|----------|-----|-------------|-------|
| id | SERIAL | PRIMARY KEY | |
| book_item_id | INTEGER | NOT NULL, UNIQUE | FOREIGN KEY REFERENCES book_items(id) |
| user_id | INTEGER | NOT NULL | FOREIGN KEY REFERENCES users(id) |
| rented_at | TIMESTAMP | DEFAULT NOW() | |
| due_date | DATE | NOT NULL | |
| returned_at | TIMESTAMP | | |
| late_fee | NUMERIC(8,2) | DEFAULT 0 | |

### Таблица `reservations`
| Имя поля | Тип | Ограничения | Связи |
|----------|-----|-------------|-------|
| id | SERIAL | PRIMARY KEY | |
| book_id | INTEGER | NOT NULL | FOREIGN KEY REFERENCES books(id) |
| user_id | INTEGER | NOT NULL | FOREIGN KEY REFERENCES users(id) |
| reserved_at | TIMESTAMP | DEFAULT NOW() | |
| expires_at | TIMESTAMP | NOT NULL | |
| status | reservation_status | DEFAULT 'Active' | |
| UNIQUE(book_id, user_id) WHERE status = 'Active' | | | |

### Таблица `reviews`
| Имя поля | Тип | Ограничения | Связи |
|----------|-----|-------------|-------|
| id | SERIAL | PRIMARY KEY | |
| rating | SMALLINT | NOT NULL, CHECK (rating >= 1 AND rating <= 5) | |
| comment | TEXT | | |
| book_id | INTEGER | NOT NULL | FOREIGN KEY REFERENCES books(id) |
| user_id | INTEGER | NOT NULL | FOREIGN KEY REFERENCES users(id) |
| created_at | TIMESTAMP | DEFAULT NOW() | |
| UNIQUE(book_id, user_id) | | | |

### Таблица `user_action_logs`
| Имя поля | Тип | Ограничения | Связи |
|----------|-----|-------------|-------|
| id | SERIAL | PRIMARY KEY | |
| user_id | INTEGER | NOT NULL | FOREIGN KEY REFERENCES users(id) |
| action_type | VARCHAR(100) | NOT NULL | |
| entity_id | INTEGER | | |
| timestamp | TIMESTAMP | DEFAULT NOW() | |

### Таблица `book_author` (MTM)
| Имя поля | Тип | Ограничения | Связи |
|----------|-----|-------------|-------|
| book_id | INTEGER | NOT NULL | FOREIGN KEY REFERENCES books(id) ON DELETE CASCADE |
| author_id | INTEGER | NOT NULL | FOREIGN KEY REFERENCES authors(id) ON DELETE CASCADE |
| PRIMARY KEY (book_id, author_id) | | | |

### Таблица `book_genre` (MTM)
| Имя поля | Тип | Ограничения | Связи |
|----------|-----|-------------|-------|
| book_id | INTEGER | NOT NULL | FOREIGN KEY REFERENCES books(id) ON DELETE CASCADE |
| genre_id | INTEGER | NOT NULL | FOREIGN KEY REFERENCES genres(id) ON DELETE CASCADE |
| PRIMARY KEY (book_id, genre_id) | | | |

## Схема базы данных

<img width="1315" height="985" alt="Database scheme" src="" />
