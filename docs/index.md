# Predict Contracts

## Обзор проекта

**Predict Contracts** - это система смарт-контрактов на блокчейне TON для создания и управления предсказательными рынками. Пользователи могут делать ставки на исходы событий, покупая токены "YES" или "NO", используя USDT в качестве базовой валюты.

### Основные возможности:
- Создание предсказательных событий через Factory контракт
- Покупка/продажа токенов YES/NO за USDT
- Автоматическое ценообразование на основе LMSR (Logarithmic Market Scoring Rule)
- Управление ликвидностью
- Система комиссий
- Определение победителя события

## Архитектура системы

### Основные компоненты

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Factory       │    │     Event       │    │  EventWallet    │
│   Contract      │───▶│   Contract      │───▶│   Contract      │
│                 │    │                 │    │                 │
│ - Создает       │    │ - Управляет     │    │ - Хранит токены │
│   события       │    │   торговлей     │    │   пользователя  │
│ - Настройки     │    │ - LMSR цены     │    │ - Покупка/продажа│
│   комиссий      │    │ - Ликвидность   │    │ - Передачи      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   USDT Minter   │    │   Math Module   │    │   Utils &       │
│   (Jetton)      │    │   (LMSR)        │    │   Constants     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## Структура проекта

### 📁 contracts/
Содержит исходный код смарт-контрактов на FunC:

#### Основные контракты:
- **`factory.fc`** - Фабрика для создания событий
- **`event.fc`** - Основной контракт события
- **`event-wallet.fc`** - Кошелек для токенов пользователя

#### Математические модули:
- **`math/predict-math-core.fc`** - Ядро LMSR алгоритма
- **`math/predict-math.fc`** - Высокоуровневые математические функции, использующие контекст контракта (c4)

#### Утилиты:
- **`utils/`** - Вспомогательные функции
- **`op-codes.fc`** - Коды операций
- **`errors.fc`** - Коды ошибок
- **`storage.fc`** - Структуры данных

### 📁 wrappers/
TypeScript обертки для взаимодействия с контрактами:

- **`Factory.ts`** - Обертка для Factory контракта
- **`Event.ts`** - Обертка для Event контракта  
- **`EventWallet.ts`** - Обертка для EventWallet контракта
- **`Math.ts`** - Обертка для математических функций

### 📁 scripts/
Скрипты для развертывания и взаимодействия:

#### Factory скрипты:
- **`deployFactory.ts`** - Развертывание фабрики
- **`deployEvent.ts`** - Создание нового события
- **`setEnable.ts`** - Включение/отключение фабрики

#### Event скрипты:
- **`addLiquidity.ts`** - Добавление ликвидности
- **`withdrawLiquidity.ts`** - Вывод ликвидности
- **`sendBuyStock.ts`** - Покупка токенов
- **`sendSellStock.ts`** - Продажа токенов
- **`setEnableTrading.ts`** - Включение торговли
- **`defineWinner.ts`** - Определение победителя
- **`getPredictData.ts`** - Получение данных события

### 📁 tests/
Тесты для всех компонентов:

- **`Predict.spec.ts`** - Основные тесты системы
- **`Math.spec.ts`** - Тесты математических функций

### 📁 utils/
Утилиты и константы:

- **`nanoUSDT.ts`** - Конвертация USDT в nanoUSDT и обратно
- **`constants.ts`** - Константы проекта
- **`crc32.ts`** - CRC32 хеширование
- **`sha256.ts`** - SHA256 хеширование
- **`tokenMetadata.ts`** - Метаданные токенов

## Детальное описание компонентов

### 1. Factory Contract

**Назначение**: Центральный контракт для создания и управления событиями.

**Основные функции**:
- Создание новых событий
- Управление настройками комиссий
- Включение/отключение системы

**Ключевые данные**:
```func
global int      ctx_seed;                    // Seed для генерации адресов
global int      ctx_is_enabled;              // Включена ли фабрика
global slice    ctx_admin;                   // Адрес администратора
global slice    ctx_fee_address;             // Адрес для комиссий
global int      ctx_buy_fee_numerator;       // Числитель комиссии за покупку
global int      ctx_sell_fee_numerator;      // Числитель комиссии за продажу
global int      ctx_trade_fee_denominator;   // Знаменатель комиссий
global cell     ctx_event_code;              // Код контракта события
global cell     ctx_wallet_code;             // Код кошелька токенов
```

**Основные операции**:
- `deploy_event` - Создание нового события
- `enable_factory` - Включение/отключение фабрики

### 2. Event Contract

**Назначение**: Управление конкретным предсказательным событием.

**Основные функции**:
- Торговля токенами YES/NO
- Управление ликвидностью
- Расчет цен по LMSR
- Определение победителя

**Ключевые данные**:
```func
global int ctx_initial_liq;        // Начальная ликвидность
global int ctx_current_liq;        // Текущая ликвидность
global int ctx_q1_supply;          // Количество токенов YES
global int ctx_q2_supply;          // Количество токенов NO
global int ctx_winner;             // Победитель (если определен)
global int ctx_trading_enabled;    // Включена ли торговля
```

**Основные операции**:
- `buy` - Покупка токенов
- `sell` - Продажа токенов
- `add_liquidity` - Добавление ликвидности
- `withdraw_liquidity` - Вывод ликвидности
- `select_winner` - Определение победителя

### 3. EventWallet Contract

**Назначение**: Кошелек пользователя для хранения токенов события.

**Основные функции**:
- Хранение токенов YES/NO пользователя
- Обработка переводов токенов
- Продажа токенов обратно в событие

**Ключевые данные**:
```func
int yes_or_no;                    // Тип токена (YES/NO)
int balance;                      // Баланс токенов
slice owner_address;              // Адрес владельца
slice jetton_master_address;      // Адрес мастер-контракта
```

### 4. Математический модуль (LMSR)

**Logarithmic Market Scoring Rule** - алгоритм для автоматического ценообразования.

**Основные функции**:

#### `calc_token_price(b, q1, q2)`
Расчет текущей цены токена:
```
price = exp(q1/b) / (exp(q1/b) + exp(q2/b))
```

#### `calc_coins_for_usdt(usdt_amount, b, q1, q2)`
Расчет количества токенов за USDT:
```
coins = b * ln(exp(usdt/b) + exp((q2-q1)/b) * (exp(usdt/b) - 1))
```

#### `calc_usdt_for_coins(token_amount, b, q1, q2)`
Расчет USDT за токены (обратная операция).

## Взаимодействие с системой

### 1. Создание события

```typescript
// 1. Развертывание фабрики
const factory = Factory.createFromConfig({
    admin: adminAddress,
    feeAddress: feeAddress,
    buyFeeNumerator: 1,
    sellFeeNumerator: 1,
    feeDenominator: 100,
    isEnabled: true,
    // ... другие параметры
}, factoryCode);

// 2. Создание события
await factory.sendDeployEvent(sender, {
    name: "Bitcoin price > $100k by 2024",
    description: "Will Bitcoin reach $100,000 by end of 2024?",
    extraMetadata: {}
});
```

### 2. Добавление ликвидности

```typescript
// Администратор добавляет начальную ликвидность
await event.sendAddLiquidity(sender, {
    liquidityAmount: toNanoUSDT("1000") // 1000 USDT
}, usdtMinterAddress);
```

### 3. Покупка токенов

```typescript
// Пользователь покупает токены YES
await event.sendBuy(sender, {
    yesOrNo: true,                    // YES токены
    usdtAmount: toNanoUSDT("100"),    // 100 USDT
    minReceive: 0n,                   // Минимальное количество токенов
}, usdtMinterAddress);
```

### 4. Продажа токенов

```typescript
// Получение кошелька пользователя
const wallet = await event.getUserWallet(provider, userAddress, true);

// Продажа токенов
await wallet.sendSellCoins(sender, {
    amount: 1000n,        // Количество токенов
    minReceive: 0n,       // Минимальная сумма USDT
});
```

### 5. Определение победителя

```typescript
// Администратор определяет победителя
await event.sendSelectWinner(sender, {
    yesOrNo: true  // YES выиграло
});
```

## Система комиссий

### Структура комиссий:
- **Комиссия за покупку**: `buyFeeNumerator / feeDenominator`
- **Комиссия за продажу**: `sellFeeNumerator / feeDenominator`
- **По умолчанию**: 1% (1/100)

### Распределение комиссий:
- Комиссии отправляются на `feeAddress`
- Комиссии вычитаются из суммы операции
- Остаток идет на торговлю

## Безопасность и ограничения

### Проверки безопасности:
- Только администратор может управлять системой
- Торговля отключена до добавления ликвидности
- Проверка минимальных сумм
- Защита от переполнения

### Коды ошибок:
- `707` - Ошибка проскальзывания
- `708` - Слишком много токенов для покупки
- `711` - Торговля отключена
- `715` - Не администратор
- `717` - Победитель уже определен
- `801` - Фабрика отключена

## Утилиты и вспомогательные функции

### nanoUSDT конвертация:
```typescript
// Конвертация USDT в nanoUSDT (1 USDT = 1,000,000 nanoUSDT)
const nanoUSDT = toNanoUSDT("100.5");  // 100500000n

// Обратная конвертация
const usdt = fromNanoUSDT(100500000n); // 100.5
```

### Константы:
```typescript
export const Constants = {
    EventDeploymentGas: toNano('0.05'),  // Газ для развертывания события
    CoinBuyGas: toNano('0.08'),          // Газ для покупки
    CoinSellGas: toNano('0.08'),         // Газ для продажи
};
```

## Тестирование

### Запуск тестов:
```bash
npm test
# или
yarn test
```

### Структура тестов:
- **Predict.spec.ts** - Интеграционные тесты всей системы
- **Math.spec.ts** - Тесты математических функций

### Пример теста:
```typescript
it('should allow buying and selling tokens', async () => {
    // Добавление ликвидности
    await event.sendAddLiquidity(deployer.getSender(), {
        liquidityAmount: toNanoUSDT("1000")
    }, USDTMinter.address);

    // Покупка токенов
    await event.sendBuy(trader.getSender(), {
        yesOrNo: true,
        usdtAmount: toNanoUSDT("100"),
        minReceive: 0n
    }, USDTMinter.address);

    // Проверка результата
    const wallet = await event.getUserWallet(provider, trader.address, true);
    const balance = (await wallet.getData()).balance;
    expect(balance).toBeGreaterThan(0n);
});
```

## Развертывание

### Запуск скриптов:
```bash
npm run start
# или
yarn start
```

### Доступные скрипты:
- `deployFactory` - Развертывание фабрики
- `deployEvent` - Создание события
- `addLiquidity` - Добавление ликвидности
- `sendBuyStock` - Покупка токенов
- `sendSellStock` - Продажа токенов
- `setEnableTrading` - Управление торговлей
- `defineWinner` - Определение победителя

## Заключение

Система Predict Contracts предоставляет полнофункциональную платформу для создания предсказательных рынков на блокчейне TON. Использование LMSR алгоритма обеспечивает справедливое ценообразование, а модульная архитектура позволяет легко расширять функциональность.

### Ключевые преимущества:
- ✅ Автоматическое ценообразование
- ✅ Справедливое распределение рисков
- ✅ Низкие комиссии
- ✅ Прозрачность операций
- ✅ Децентрализованное управление
- ✅ Масштабируемость

### Возможности расширения:
- Поддержка множественных исходов
- Реферальная система
- Автоматическое определение победителей
- Торговля через другие токены, не только USDT
