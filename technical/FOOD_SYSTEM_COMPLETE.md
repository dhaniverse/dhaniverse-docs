# Dhaniverse Food System - Complete Documentation

## Overview
The Dhaniverse food system is a comprehensive survival mechanic that simulates hunger, energy, and mental state through food consumption, buff management, and stat tracking. Players must balance their nutrition, manage stamina, and strategically use consumables to optimize their performance.

---

## Core Stats

### Primary Stats
| Stat | Max Value | Description |
|------|-----------|-------------|
| **Health (HP)** | 100 | Physical and mental well-being. 0 HP = burnout/forced rest |
| **Food Bar** | 100% | Satiation level. Decays over time |
| **Stamina** | 100 | Energy for running and tasks. Drains when running |
| **Level** | ∞ | Player progression level |
| **XP** | Variable | Experience points for leveling up |

### Secondary "Soft" Stats (Influenced by Buffs/Debuffs)
| Stat | Base Value | Description |
|------|------------|-------------|
| **Focus** | 0 | Success rate for studying, assignments, mini-games |
| **Creativity** | 0 | Success rate for art, music, problem-solving |
| **Social** | 0 | Success rate for conversations, networking |
| **Movement Speed** | 100% | Character traversal speed |
| **Efficiency** | 100% | Task completion and resource gathering speed |

---

## Food Bar Mechanics

### Decay System
- **Decay Rate**: 100% → 0% in **12 minutes** (~0.139% per second)
- **Decay Tick**: Updates every **1 second**
- **Auto-starts**: When stats become visible (after Maya tutorial)

### Food Bar States

#### 🍔 Normal (20-89%)
- **Effects**: Optimal performance
- **Movement Speed**: 100% (base)
- **No debuffs**

#### 😰 Hungry (0-19%)
- **Effects**:
  - **-10% Movement Speed** (90%)
  - **-5 Focus**
  - **-5 Creativity**
  - **Lose 1 HP every 10 seconds**
- **Visual**: Red/warning indicators
- **Recovery**: Eat food to restore Food Bar above 20%

#### 🤢 Stuffed (90-100%)
- **Effects**:
  - **-15% Movement Speed** (85%)
  - **+5% "Bloated" Debuff** (lasts 5 minutes)
    - **-5 Social** (reduced interaction success rate)
  - **Eating while stuffed**: Minor HP loss (-5 HP per item)
- **Visual**: Bloated/green indicator
- **Recovery**: Wait for Food Bar to drop below 90%

---

## Stamina System

### Drain Timing
- **Total Drain Time**: **3 minutes** (180 seconds) to fully exhaust
- **Drain Rate**: ~0.555% per second when running
- **Recovery Rate**: 20% per second (5 seconds to fully recover)
- **Recovery Delay**: 500ms after stopping running
- **Minimum to Run**: 5% stamina required

### Running Mechanics
1. **Hold Shift**: Starts running (if stamina ≥ 5%)
2. **Stamina depletes**: At ~0.555% per second
3. **Stamina hits 0%**: Running stops automatically
4. **Depletion Flag Set**: Player MUST wait for **full recovery** (100%) before running again
5. **Recovery**: Starts 500ms after releasing Shift, recovers at 20%/sec

### Special Cases
- **Stats Not Visible**: Unlimited stamina (tutorial mode)
- **Depleted State**: Cannot run until stamina fully recovers to 100%

---

## Food Items Catalog

### 🍴 Food Items

| Item | Price (₹) | HP | Food Bar | Stamina | Buffs/Debuffs | Rarity |
|------|-----------|-----|----------|---------|--------------|--------|
| **A Single, Sad Bean** | 5 | +1 | +2% | 0 | None | Common |
| **Apple** | 15 | +5 | +5% | 0 | None | Common |
| **Berries** | 20 | 0 | +8% | +10 | None | Common |
| **Pizza Rolls** | 30 | +25 | +15% | 0 | None | Common |
| **Taco** | 45 | +20 | +20% | +10 | None | Uncommon |
| **Avocado Toast** | 60 | +50 | +25% | 0 | None | Uncommon |
| **Plant-Based Burger** | 70 | +40 | +30% | 0 | +15 Social (10 min) | Uncommon |
| **Upgraded Instant Ramen** | 80 | +40 | +20% | +20 | None | Uncommon |
| **Girl Dinner** | 90 | Random 20-60 | +35% | 0 | None (Chaotic HP) | Rare |
| **The Viral Feta Pasta** | 150 | **FULL HP** | +50% | 0 | +10 Social, +10 Creativity (15 min) | Legendary |

### ☕ Drink Items

| Item | Price (₹) | HP | Food Bar | Stamina | Buffs/Debuffs | Rarity |
|------|-----------|-----|----------|---------|--------------|--------|
| **Iced Coffee / Cold Brew** | 45 | 0 | +5% | +40 | +20 Movement Speed (2 min) | Uncommon |
| **Boba Tea** | 50 | 0 | +10% | **FULL** | +10 Focus (10 min) | Uncommon |
| **Energy Drink (Glitch Fuel)** | 75 | 0 | +10% | +60 | +20 Efficiency (5 min) → -10 Stamina Crash (5 min after) | Rare |
| **Acai Bowl** | 85 | 0 | +20% | +20 | +2 HP/sec Regen (10 sec) | Rare |
| **Flamin' Hot Cheetos** | 40 | **-5** | +15% | 0 | +15 Creativity (5 min) | Uncommon |

---

## Consumption Text (Toast Messages)

Each item displays a unique message when consumed:

**Examples:**
- **Single Bean**: "It's just one bean. What did you expect? Still, it's something. (+1HP)"
- **Iced Coffee**: "You now feel capable of starting that assignment you've been putting off for three weeks. ZOOM! (+40 Stamina, +Speed)"
- **Viral Feta Pasta**: "The dish that launched a thousand TikToks. Warms the soul and protects from bad vibes. (+Full HP, +Comforted)"

Toast animations fade in/out with rarity-based colors and styling.

---

## Pricing & Economy

### Tax System
- **Food Tax**: **2%** on all purchases
- **Calculation**: `tax = Math.round(subtotal * 0.02)`
- **Total Cost**: `subtotal + tax`

### Example Purchase
- **Item**: Boba Tea (₹50)
- **Quantity**: 2
- **Subtotal**: ₹100
- **Tax (2%)**: ₹2
- **Total**: **₹102**

### Balance Philosophy
- **Value Proposition**: Items priced by combined HP/Food/Stamina/Buff value
- **Strategic Consumption**: Players weigh cost vs. current needs
- **Food Bar Constraint**: Prevents cheap item spamming (stuffed state penalty)
- **Time is Money**: Time-limited buffs encourage strategic pre-task usage

---

## Technical Implementation

### File Structure
```
client/src/
├── services/
│   ├── PlayerStatsManager.ts         # Core stat system with food mechanics
│   └── FoodInventoryManager.ts       # Inventory tracking
├── config/
│   └── FoodCatalog.ts                # All food/drink item definitions
├── types/
│   └── FoodTypes.ts                  # TypeScript interfaces
├── ui/components/
│   ├── food/
│   │   ├── FoodShopDashboard.tsx     # Purchase UI
│   │   ├── FoodConsumptionToast.tsx  # Consumption feedback
│   │   └── Inventory.tsx             # Inventory management
│   └── hud/GameHUD.tsx               # Stats display
└── game/systems/
    └── BuildingManager.ts            # Food shop entrance [9475, 4546]
```

### Backend API Endpoints
```typescript
// Get inventory
GET /game/food-inventory
Response: { success: true, data: FoodInventoryDocument }

// Purchase items
POST /game/food/purchase
Body: { items: [{itemId, quantity}], totalCost: number }
Response: { success: true, data: { inventory, newBalance } }

// Consume item
POST /game/food/consume
Body: { itemId: string, statsApplied: {hp, foodBar, stamina} }
Response: { success: true, data: { inventory } }
```

### Database Schema (MongoDB)
```typescript
FoodInventoryDocument {
  userId: string;
  items: [
    {
      itemId: string;
      quantity: number;
      acquiredAt: Date;
    }
  ];
  lastUpdated: Date;
}

FoodTransactionDocument {
  userId: string;
  type: 'purchase' | 'consume';
  itemId: string;
  quantity: number;
  totalCost?: number;
  statsApplied?: object;
  timestamp: Date;
}
```

---

## PlayerStatsManager API

### Key Methods
```typescript
// Stats Management
playerStatsManager.getStats(): PlayerStats
playerStatsManager.updateStat(stat, value): void
playerStatsManager.setFood(value): void
playerStatsManager.setHealth(value): void

// Food Consumption
playerStatsManager.consumeFood(
  hpRestored: number | 'full' | 'random',
  foodBarIncrease: number,
  staminaRestored: number | 'full',
  buffs?: Buff[],
  debuffs?: Debuff[]
): void

// Buff/Debuff Management
playerStatsManager.addBuff(type, value, duration): void
playerStatsManager.addDebuff(type, value, duration): void
playerStatsManager.getActiveBuffs(): Buff[]
playerStatsManager.getActiveDebuffs(): Debuff[]

// Stamina Control
playerStatsManager.startRunning(): boolean
playerStatsManager.stopRunning(): void
playerStatsManager.canPlayerRun(): boolean

// System Control
playerStatsManager.setStatsVisible(visible: boolean): void
playerStatsManager.getStatsVisible(): boolean
playerStatsManager.reset(): void
playerStatsManager.destroy(): void
```

### Buff/Debuff Interface
```typescript
interface Buff {
  type: 'focus' | 'creativity' | 'social' | 'movement_speed' | 'efficiency' | 'hp_regen';
  value: number;           // Positive value
  duration: number;        // In milliseconds
  startTime: number;       // Timestamp
}

interface Debuff {
  type: 'hunger' | 'bloated' | 'sugar_crash';
  value: number;           // Negative value
  duration: number;
  startTime: number;
}
```

---

## Food Shop Location

### Coordinates
- **X**: 9475
- **Y**: 4546
- **Interaction Distance**: Standard building interaction range
- **Trigger Key**: **E** key when near entrance

### Access Requirements
- **Stats must be visible**: Unlocked after completing Maya's stamina tutorial
- **Before unlock**: Shows dialogue "Unlock your stats from Maya first to access the Food Shop!"

### Opening Event
```typescript
window.addEventListener('openFoodShop', () => {
  // Open FoodShopDashboard component
});
```

---

## Gameplay Loop

### 1. Food Decay Phase (12 minutes)
```
Food 100% → 89%  : Normal performance
Food 89% → 20%   : Optimal range
Food 20% → 0%    : Hungry debuffs + HP drain
```

### 2. Purchase Decision
- **Check balance**: Current ₹ rupees
- **Assess needs**: HP? Stamina? Buff for upcoming task?
- **Buy items**: Pay with 2% tax
- **Items added to inventory**

### 3. Consumption Strategy
- **Monitor Food Bar**: Don't eat while stuffed (90-100%)
- **Time buffs**: Consume before important events (exams, presentations)
- **Balance stats**: HP vs Stamina vs Buffs
- **Watch duration**: Buff timers in HUD

### 4. Buff Management
- **Active buffs displayed**: UI shows remaining time
- **Stack buffs**: Multiple buffs can be active
- **Debuff awareness**: Sugar crash after energy drinks
- **Comforted state**: Social + Creativity boost from Feta Pasta

---

## Visual Feedback

### Toast Notifications (FoodConsumptionToast)
- **Common**: Gray border, standard fade
- **Uncommon**: Green border, 200ms longer animation
- **Rare**: Blue border, glow effect
- **Legendary**: Gold border, sparkle particles, extended duration

### HUD Indicators
- **Health Bar**: Red, displays HP/100
- **Food Bar**: Orange/yellow, displays %
- **Stamina Bar**: Blue, displays %
- **Buff Icons**: Top-right corner with timers
- **Debuff Icons**: Red-tinted with timers

### Stat Colors
- **Positive Buffs**: Green text (+15 Social)
- **Negative Debuffs**: Red text (-10 Movement Speed)
- **Neutral**: White/gray

---

## Advanced Mechanics

### HP Regeneration
- **Acai Bowl**: +2 HP/sec for 10 seconds = +20 HP total
- **Ticks every second**: Independent of other systems
- **Stacks with other healing**: Can eat multiple regen items

### Sugar Crash
- **Energy Drink**: +20 Efficiency for 5 minutes
- **After buff expires**: -10 Stamina "Sugar Crash" for 5 minutes
- **Debuff applied automatically**: No item needed
- **Recovery**: Wait for debuff to expire

### Random HP (Girl Dinner)
- **Range**: 20-60 HP (random)
- **Calculated on consumption**: `Math.floor(Math.random() * 41) + 20`
- **Chaotic but fun**: Risk/reward element

### Full HP Restoration
- **The Viral Feta Pasta**: Restores HP to 100%
- **Most expensive item**: ₹150 (₹153 with tax)
- **Best value for emergency**: Full heal + buffs + 50% food

---

## Game Balance

### Early Game (₹0-1000)
- **Single Bean, Apple, Berries**: Cheap survival items
- **Focus on rupee management**: Save for bank/stocks

### Mid Game (₹1000-5000)
- **Pizza Rolls, Taco, Coffee**: Balanced cost/benefit
- **Start using buffs**: Iced Coffee for speed, Boba for focus

### Late Game (₹5000+)
- **Plant Burger, Ramen, Acai Bowl**: Premium items
- **Strategic buff timing**: Pre-exam Boba Tea, pre-presentation Feta Pasta
- **Efficiency builds**: Energy Drink chains (manage crash)

---

## Debug Commands

```typescript
// In browser console (window.playerStatsManager exposed)

// Check current stats
playerStatsManager.getStats()

// Set specific stat
playerStatsManager.setFood(50)
playerStatsManager.setHealth(75)
playerStatsManager.updateStat('stamina', 100)

// Add test buff
playerStatsManager.addBuff('focus', 15, 10 * 60 * 1000) // +15 focus for 10 min

// Add test debuff
playerStatsManager.addDebuff('bloated', -5, 5 * 60 * 1000) // -5 social for 5 min

// View active effects
playerStatsManager.getActiveBuffs()
playerStatsManager.getActiveDebuffs()

// Reset everything
playerStatsManager.reset()
```

---

## Future Enhancements

### Planned Features
1. **Food Crafting**: Combine ingredients for custom items
2. **Restaurant Upgrades**: Unlock premium items with progression
3. **Daily Specials**: Rotating discount items
4. **Food Quests**: "Eat 10 different items" achievements
5. **Recipe Book**: Track discovered items
6. **Meal Plans**: Pre-purchase bundles with discounts
7. **Seasonal Items**: Limited-time foods with unique buffs
8. **Cooking Minigame**: Skill-based item creation

### Technical Improvements
1. **Save/Load**: Persist buffs/debuffs across sessions
2. **Analytics**: Track most consumed items, spending patterns
3. **Optimization**: Cache food item lookups
4. **Real-time Sync**: WebSocket updates for multiplayer food sharing
5. **Mobile UI**: Touch-optimized food shop interface

---

## Troubleshooting

### Common Issues

**Issue**: "Stats not visible, can't access food shop"
- **Cause**: Haven't completed Maya's stamina tutorial
- **Fix**: Talk to Maya after stock market onboarding

**Issue**: "Food bar not decaying"
- **Cause**: Stats visibility not enabled
- **Fix**: Complete tutorial, check `playerStatsManager.getStatsVisible()`

**Issue**: "Can't run even with stamina"
- **Cause**: Stamina was depleted, must fully recover
- **Fix**: Wait for stamina to hit 100%, then try again

**Issue**: "Purchased items not in inventory"
- **Cause**: API request failed or insufficient funds
- **Fix**: Check balance, check network tab, retry purchase

**Issue**: "Buffs not applying"
- **Cause**: Buff/debuff system not started (stats not visible)
- **Fix**: Ensure `setStatsVisible(true)` called

---

## Performance Considerations

### Update Frequencies
- **Stamina drain/recovery**: 100ms tick
- **Food decay**: 1 second tick
- **HP drain (hunger)**: 10 second tick
- **Buff/debuff check**: 1 second tick

### Optimization Tips
1. **Use intervals wisely**: All timers cleared when stats hidden
2. **Batch stat updates**: `notifyListeners()` called once per update
3. **LocalStorage**: Only save on stat change, not every tick
4. **Event throttling**: Food consumption limited by interaction cooldowns

---

## Conclusion

The Dhaniverse food system creates a survival layer that encourages strategic resource management, time planning, and decision-making. Players must balance:
- **Short-term survival** (staying fed, healthy)
- **Long-term optimization** (saving for expensive items)
- **Buff timing** (consuming before key events)
- **Economic efficiency** (maximizing value per rupee)

This system integrates seamlessly with the banking and stock market systems, creating a holistic economic simulation where every rupee matters and every choice has consequences.

**Eat smart. Play strategic. Prosper in Dhaniverse.** 🍔💰📈
