# Food System Documentation

## Overview

The Dhaniverse food system provides a comprehensive RPG-style consumables system where players can purchase food and drinks to manage their health, stamina, and food bar stats. The system includes item rarities, buffs/debuffs, and an immersive UI experience with toast notifications.

## Architecture

### 1. Core Components

#### Type Definitions (`src/types/FoodTypes.ts`)
- **ConsumableItem**: Base type for food and drink items
- **PlayerStats**: Health (HP), Food Bar, Stamina, and secondary stats (Focus, Creativity, Social, Movement Speed, Efficiency)
- **BuffEffect** & **DebuffEffect**: Temporary stat modifications
- **ActiveEffect**: Currently active buffs/debuffs with expiration
- **FoodRarity**: `common`, `uncommon`, `rare`, `legendary`

#### Food Catalog (`src/config/FoodCatalog.ts`)
Contains all available food and drink items with properties:
- **Price**: Base cost in rupees (₹)
- **HP Restored**: Can be number, `'random'` (20-60), or `'full'`
- **Food Bar Increase**: Percentage (0-100%)
- **Stamina Restored**: Can be number or `'full'`
- **Buffs**: Temporary stat bonuses with duration
- **Debuffs**: Negative effects (e.g., Sugar Crash)
- **Consumption Text**: Flavor text displayed when eaten
- **Rarity**: Visual and mechanical importance tier

**Tax Calculation**: 2% food tax applied to all purchases

### 2. Management Services

#### PlayerStatsManager (`src/services/PlayerStatsManager.ts`)
- Manages all player stats (HP, Food Bar, Stamina, etc.)
- Handles passive systems:
  - **Stamina Recovery**: +5 stamina every 5 seconds
  - **Hunger Drain**: -1 HP every 10 seconds when food bar ≤ 19%
  - **Effect Expiration**: Automatic buff/debuff removal
- **Food Bar Status Effects**:
  - **Hungry** (0-19%): -10% Movement Speed, -5% Focus, -5% Creativity
  - **Normal** (20-89%): Optimal performance
  - **Stuffed** (90-100%): -15% Movement Speed, +5% Bloated debuff

#### FoodInventoryManager (`src/services/FoodInventoryManager.ts`)
- Manages player food inventory
- Handles item consumption with stat modifications
- Dispatches events for UI updates
- Integrates with `PlayerStatsManager` for stat changes

### 3. UI Components

#### FoodShopDashboard (`src/ui/components/food/FoodShopDashboard.tsx`)
**Location Trigger**: `[9475, 4546]` (outdoor world location)

**Features**:
- Tabbed interface (Food / Drinks & Snacks)
- Search functionality
- Shopping cart with quantity management
- Real-time balance checking
- Rarity-based visual styling
- Tax calculation display
- Item stats preview

**Purchase Flow**:
1. Player adds items to cart
2. System calculates: Subtotal + Tax (2%) = Total
3. Checks player rupees via `balanceManager`
4. Deducts rupees and adds items to inventory
5. Shows success notification

#### FoodConsumptionToast (`src/ui/components/food/FoodConsumptionToast.tsx`)
Displays immersive consumption notifications:
- **Rarity-based styling**: Different colors, borders, glows
- **Animation duration**: Based on rarity (3-6 seconds)
- **Fade animations**: Smooth entry/exit
- **Legendary effects**: Special pulsing glow animation

#### EnhancedInventory (`src/ui/components/food/EnhancedInventory.tsx`)
- Displays owned food/drink items
- Shows item stats and quantity
- Click-to-consume interface
- Rarity-based visual presentation
- Real-time inventory updates

### 4. Game Integration

#### BuildingManager Update (`src/game/systems/BuildingManager.ts`)
Added `handleFoodShopInteraction()`:
- **Entrance Location**: `{ x: 9475, y: 4546 }`
- **Interaction Distance**: Uses `Constants.BUILDING_INTERACTION_DISTANCE`
- **Trigger Key**: `E` key
- **Event Dispatch**: `'openFoodShop'` window event

**Interaction Flow**:
1. Player approaches location [9475, 4546]
2. Interaction prompt appears: "Press [E] to Enter Dhani's Food Corner"
3. Player presses `E`
4. Food shop UI opens
5. Player can shop and close UI with `X` button

## Game Mechanics

### Health System
- **Max HP**: 100
- **Game Over**: HP reaches 0 (burnt out/stressed out)
- **Recovery**: Consuming food items
- **Passive Drain**: -1 HP every 10 seconds when hungry

### Food Bar System
- **Max**: 100%
- **Prevents Spam**: Even cheap items increase food bar
- **Overeating Penalty**: -2 HP when eating while stuffed (90-100%)
- **Hunger Effects**: Applied when <20%

### Stamina System
- **Max**: 100
- **Usage**: Running, studying, creative work, social events
- **Exhaustion**: 0 stamina = cannot perform stamina actions, -5% movement speed
- **Recovery Rate**: +5 every 5 seconds (passive)
- **Day Cycle**: Stamina exhausts after 3 minutes (24-minute day = 1 game day)

### Secondary Stats (Influenced by Buffs/Debuffs)
- **Focus**: Success rate for studying, assignments, mini-games
- **Creativity**: Success rate for art, music, problem-solving
- **Social**: Success rate for conversations, networking
- **Movement Speed**: Map traversal speed
- **Efficiency**: Task completion speed

## Item Examples

### Food Items

| Item | Price (₹) | HP | Food | Stamina | Buffs | Rarity |
|------|-----------|-----|------|---------|-------|--------|
| A Single, Sad Bean | 5 | +1 | +2% | 0 | None | Common |
| Apple | 15 | +5 | +5% | 0 | None | Common |
| Pizza Rolls | 30 | +25 | +15% | 0 | None | Common |
| Taco | 45 | +20 | +20% | +10 | None | Uncommon |
| Plant-Based Burger | 70 | +40 | +30% | 0 | +15 Social (10min) | Uncommon |
| Girl Dinner | 90 | Random 20-60 | +35% | 0 | None | Rare |
| Viral Feta Pasta | 150 | Full | +50% | 0 | +10 Social, +10 Creativity (15min) | Legendary |

### Drink Items

| Item | Price (₹) | HP | Food | Stamina | Buffs | Rarity |
|------|-----------|-----|------|---------|-------|--------|
| Iced Coffee | 45 | 0 | +5% | +40 | +20 Movement Speed (2min) | Uncommon |
| Boba Tea | 50 | 0 | +10% | Full | +10 Focus (10min) | Uncommon |
| Energy Drink | 75 | 0 | +10% | +60 | +20 Efficiency (5min), then -10 Stamina crash | Rare |
| Flamin' Hot Cheetos | 40 | -5 | +15% | 0 | +15 Creativity (5min) | Uncommon |

## Event System

### Dispatched Events

**From FoodInventoryManager**:
```typescript
// Consumption event
window.dispatchEvent(new CustomEvent('food-consumed', {
  detail: {
    itemName: string,
    consumptionText: string,
    rarity: string,
    hpRestored: number,
    staminaRestored: number,
    foodBarIncrease: number
  }
}));

// Inventory update
window.dispatchEvent(new CustomEvent('inventory-updated', {
  detail: { inventory: InventoryItem[] }
}));
```

**From PlayerStatsManager**:
```typescript
// Stats update
window.dispatchEvent(new CustomEvent('player-stats-updated', {
  detail: PlayerStats
}));

// Buff applied
window.dispatchEvent(new CustomEvent('buff-applied', {
  detail: { type, value, duration, itemName }
}));

// Debuff applied
window.dispatchEvent(new CustomEvent('debuff-applied', {
  detail: { type, value, duration, itemName }
}));
```

**From BuildingManager**:
```typescript
// Open food shop
window.dispatchEvent(new CustomEvent('openFoodShop'));
```

## Database Schema (Backend)

### Food Inventory Collection
```typescript
interface FoodInventoryDocument {
  userId: string;
  items: {
    itemId: string;
    quantity: number;
    type: 'food' | 'drink';
  }[];
  lastUpdated: Date;
}
```

### Food Transactions Collection
```typescript
interface FoodTransactionDocument {
  userId: string;
  itemId: string;
  itemName: string;
  quantity: number;
  totalPrice: number;
  tax: number;
  type: 'food' | 'drink';
  timestamp: Date;
}
```

### Player Stats Extension
Add to existing `PlayerStateDocument`:
```typescript
interface PlayerStateDocument {
  // ... existing fields
  stats: {
    health: number; // 0-100
    foodBar: number; // 0-100%
    stamina: number; // 0-100
    focus: number;
    creativity: number;
    social: number;
    movementSpeed: number;
    efficiency: number;
    activeBuffs: ActiveEffect[];
    activeDebuffs: ActiveEffect[];
  };
}
```

## Backend API Endpoints

### GET `/game/food-inventory`
Returns player's food inventory.

**Response**:
```json
{
  "success": true,
  "data": {
    "items": [
      { "itemId": "apple", "quantity": 5, "type": "food" }
    ]
  }
}
```

### POST `/game/food/purchase`
Purchase food items.

**Request**:
```json
{
  "items": [
    { "itemId": "apple", "quantity": 3 }
  ]
}
```

**Response**:
```json
{
  "success": true,
  "totalCost": 46, // including 2% tax
  "newBalance": 99954
}
```

### POST `/game/food/consume`
Consume a food item (optional - can be client-side only).

**Request**:
```json
{
  "itemId": "apple"
}
```

### GET `/game/player-stats`
Get current player stats.

**Response**:
```json
{
  "success": true,
  "stats": {
    "health": 85,
    "foodBar": 65,
    "stamina": 90,
    "focus": 100,
    "creativity": 100,
    "social": 115,
    "movementSpeed": 100,
    "efficiency": 100,
    "activeBuffs": [
      {
        "id": "social_1234567890",
        "type": "social",
        "value": 15,
        "expiresAt": 1234567890,
        "name": "Plant-Based Burger"
      }
    ],
    "activeDebuffs": []
  }
}
```

## Integration Steps

### 1. Add Components to GameHUD
```tsx
// GameHUD.tsx
import FoodShopDashboard from '../food/FoodShopDashboard';
import FoodConsumptionToast from '../food/FoodConsumptionToast';
import EnhancedInventory from '../food/EnhancedInventory';

const GameHUD = () => {
  const [isFoodShopOpen, setIsFoodShopOpen] = useState(false);
  const [isInventoryOpen, setIsInventoryOpen] = useState(false);

  useEffect(() => {
    const handleOpenFoodShop = () => setIsFoodShopOpen(true);
    window.addEventListener('openFoodShop', handleOpenFoodShop);
    return () => window.removeEventListener('openFoodShop', handleOpenFoodShop);
  }, []);

  return (
    <>
      {/* Existing HUD elements */}
      <FoodConsumptionToast />
      {isFoodShopOpen && (
        <FoodShopDashboard onClose={() => setIsFoodShopOpen(false)} />
      )}
      <EnhancedInventory 
        isOpen={isInventoryOpen}
        onClose={() => setIsInventoryOpen(false)}
      />
    </>
  );
};
```

### 2. Add Backend Routes
See `gameRouter.ts` - add food inventory and purchase routes.

### 3. Testing
- Navigate to `[9475, 4546]` in game
- Press `E` to open food shop
- Purchase items
- Open inventory (`I` key)
- Consume items
- Observe stat changes and toast notifications

## Balancing Philosophy

- **Value Proposition**: Items priced by HP + Food + Stamina + Buffs value
- **Strategic Consumption**: Players manage food bar to avoid overeating
- **Time is Money**: Buffs are time-limited, encouraging strategic use before important events
- **No Spam**: Food bar constraint prevents cheap item abuse
- **Risk/Reward**: Some items (Hot Cheetos) have negative HP for positive buffs

## Future Enhancements

1. **Cooking System**: Combine ingredients to craft custom foods
2. **Restaurants**: NPCs selling exclusive items
3. **Food Quests**: Special missions for rare recipes
4. **Seasonal Items**: Limited-time foods during events
5. **Status Effects UI**: Visual indicators for active buffs/debuffs
6. **Recipe Book**: Collection system for discovered foods
7. **Multiplayer Sharing**: Trade or gift food items
8. **Hunger Animation**: Visual cues when food bar is low

## Conclusion

The food system adds depth to Dhaniverse by introducing resource management, strategic planning, and meaningful player choices. The combination of stat tracking, economic cost, and immersive UI creates an engaging non-combat challenge system that complements the financial education core gameplay.
