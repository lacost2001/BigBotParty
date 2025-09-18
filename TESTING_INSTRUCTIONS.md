# ИНСТРУКЦИЯ ПО ТЕСТИРОВАНИЮ ИСПРАВЛЕНИЙ

## Как проверить, работает ли добавление участников

### 1. Запуск бота
```bash
cd "c:\Users\Administrator\Desktop\Новая папка (2)"
& ".venv\Scripts\python.exe" "Bigbot\bot_main.py"
```

### 2. Проверка состояния сессий
В Discord используйте команду: `/debug_sessions`
- Должно показать активные сессии (если есть)
- Или сообщение "Нет активных сессий"

### 3. Создание тестовой заявки
1. Используйте команду: `/unified_events_panel`
2. Выберите любое событие (например, "🪙 Золотая сфера - доставка")
3. Бот создаст тред и отправит сообщение с инструкциями

### 4. Тестирование добавления участников
В созданном треде попробуйте разные варианты:

**Вариант 1: Только я**
```
только я
```

**Вариант 2: Упоминания пользователей**
```
@user1 @user2 @user3
```

**Вариант 3: Смешанный вариант**
```
Участники: @user1 @user2
```

### 5. Что должно произойти
1. Бот должен отреагировать на сообщение
2. Показать embed "✅ Участники добавлены"
3. Перечислить всех участников
4. Показать кнопку "✅ Подтвердить заявку"

### 6. Диагностика проблем

#### Если участники не добавляются:
1. Проверьте логи бота в консоли
2. Ищите сообщения с тегами:
   - `[ON_MESSAGE]`
   - `[PARTICIPANTS START]`
   - `[PARTICIPANTS PARSED]`
   - `[HANDLE SESSION]`

#### Если сессия не найдена:
1. Используйте `/debug_sessions` для проверки активных сессий
2. Убедитесь, что ID треда соответствует ключу сессии
3. Проверьте, что пользователь - автор заявки

#### Дополнительные команды для диагностики:
- `/test_debug` - общая диагностика бота
- `/debug_sessions` - проверка активных сессий

### 7. Примеры логов при правильной работе

При отправке сообщения с упоминаниями должны появиться такие логи:
```
[ON_MESSAGE] handle_submission_message returned: True for message: '@user1 @user2'
[PARTICIPANTS START] key=123456789_987654321 msg_author=123456789 content='@user1 @user2' state=waiting_participants
[PARTICIPANTS CHECK] has_mentions=True has_keywords=False mentions_count=2
[PARTICIPANTS PARSED] count=3 ids=[123456789, 111111111, 222222222]
```

### 8. Если ничего не работает

Убедитесь в:
1. **Права бота** - может ли он читать сообщения и упоминания в треде
2. **Настройка интентов** - включен ли `message_content` intent
3. **Импорты модулей** - нет ли ошибок при загрузке ui_components
4. **Версия discord.py** - совместима ли с кодом

## Быстрый тест через команды

Можно также добавить тестовую команду для создания сессии вручную:

```python
@app_commands.command(name="create_test_session")
async def create_test_session(self, interaction: discord.Interaction):
    from .submission_state import active_submissions
    from .ui_components import InteractiveSubmissionSession
    from .events import EventType, EventAction
    
    if isinstance(interaction.channel, discord.Thread):
        session = InteractiveSubmissionSession(
            user_id=interaction.user.id,
            channel_id=interaction.channel.id,
            event_type=EventType.SPHERE_GOLD,
            action=EventAction.TRANSPORT
        )
        
        session_key = f"{interaction.user.id}_{interaction.channel.id}"
        active_submissions[session_key] = session
        
        await interaction.response.send_message(
            f"✅ Тестовая сессия создана: {session_key}", 
            ephemeral=True
        )
    else:
        await interaction.response.send_message(
            "❌ Эта команда работает только в тредах", 
            ephemeral=True
        )
```

Эта команда поможет создать сессию вручную для тестирования.
