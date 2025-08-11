# Gameplay Mechanics

## Core Game Systems

### Character System

#### Character Selection
Players choose from 4 distinct avatars (C1-C4) at the start of their journey:
- **C1**: The Entrepreneur - Focused on business and investment
- **C2**: The Saver - Emphasizes banking and compound interest
- **C3**: The Trader - Specializes in stock market activities
- **C4**: The Tech Pioneer - Web3 and cryptocurrency focused

Each character provides slight bonuses to their specialized areas but doesn't restrict access to other content.

#### Character Progression
```typescript
interface PlayerProgress {
  level: number;           // Overall player level (1-50)
  experience: number;      // XP points for leveling up
  skillPoints: number;     // Points to spend on skill trees
  achievements: string[];  // Unlocked achievements
  completedTutorials: string[];
}
```

**Level Progression:**
- **Levels 1-5**: Basic financial concepts (saving, budgeting)
- **Levels 6-15**: Investment fundamentals (stocks, bonds, risk)
- **Levels 16-25**: Advanced finance (options, derivatives, portfolio theory)
- **Levels 26-35**: Business and entrepreneurship
- **Levels 36-50**: Web3 and DeFi mastery

### Financial Systems

#### Currency System
**Dual Currency Model:**
- **Rupees**: Primary in-game currency for traditional finance
- **Tokens**: Blockchain-based currency for Web3 features

**Exchange Rate:**
- 1 Rupee = 0.1 Token (dynamic based on player activity)
- Players can exchange between currencies at the bank
- Exchange rates teach real-world forex concepts

#### Banking System
```typescript
interface BankAccount {
  balance: number;
  interestRate: number;    // Annual percentage yield
  transactions: Transaction[];
  fixedDeposits: FixedDeposit[];
}

interface FixedDeposit {
  amount: number;
  duration: number;        // in days
  interestRate: number;
  maturityDate: Date;
  status: 'active' | 'matured' | 'claimed';
}
```

**Banking Features:**
- **Savings Account**: Earn 2-5% annual interest
- **Fixed Deposits**: Higher rates (5-12%) for locked funds
- **Loans**: Borrow against assets for larger investments
- **Credit Score**: Builds based on repayment history

#### Stock Market System
```typescript
interface Stock {
  id: string;
  name: string;
  sector: string;
  currentPrice: number;
  priceHistory: number[];
  volatility: number;
  marketCap: number;
  peRatio: number;
  dividend: number;
}
```

**Market Mechanics:**
- **Real-time Price Updates**: Prices change every 30 seconds
- **Market Events**: News affects stock prices realistically
- **Sector Rotation**: Different sectors perform better at different times
- **Economic Cycles**: Bull and bear markets with realistic patterns

**Available Stocks:**
- **TECH**: Technology companies (high growth, high volatility)
- **BANK**: Financial institutions (stable, dividend-paying)
- **ENERGY**: Oil and renewable energy (cyclical)
- **HEALTH**: Healthcare and pharmaceuticals (defensive)
- **CONSUMER**: Retail and consumer goods (moderate growth)

### Building Interaction System

#### Bank Building
**Location**: Town center, easily accessible
**NPCs**: Friendly banker who explains concepts
**Services:**
- Open savings account
- Create fixed deposits
- Apply for loans
- View transaction history
- Learn about compound interest

**Interactive Elements:**
- ATM for quick transactions
- Information boards with financial tips
- Waiting area with educational content

#### Stock Exchange Building
**Location**: Financial district
**NPCs**: Professional broker with market insights
**Services:**
- Buy and sell stocks
- View market data and charts
- Research company fundamentals
- Set up automated trading rules
- Learn about investment strategies

**Interactive Elements:**
- Trading floor with live price displays
- Research terminals with company data
- News ticker with market-moving events

#### Web3 Hub (Unlocked at Level 5)
**Location**: Tech district (appears after tutorial completion)
**NPCs**: Crypto Sage - friendly guide to blockchain
**Services:**
- Convert rupees to tokens
- Connect Web3 wallet
- Stake tokens for rewards
- Participate in DeFi protocols
- Learn about blockchain technology

### Social Features

#### Multiplayer Interactions
```typescript
interface PlayerInteraction {
  type: 'chat' | 'trade' | 'challenge' | 'help';
  participants: string[];
  data: any;
  timestamp: Date;
}
```

**Chat System:**
- Global chat for all players
- Private messages between friends
- Educational discussions encouraged
- Moderation for appropriate content

**Friend System:**
- Add friends and view their progress
- Compare portfolio performance
- Share investment strategies
- Collaborative learning challenges

#### Leaderboards
**Categories:**
- **Total Wealth**: Combined rupees and token value
- **Trading Profit**: Best stock market performance
- **Savings Champion**: Highest interest earned
- **Learning Leader**: Most tutorials completed
- **Social Contributor**: Most helpful to other players

**Rewards:**
- Weekly recognition for top performers
- Special badges and achievements
- Bonus rupees for consistent performance
- Access to exclusive content and features

### Achievement System

#### Achievement Categories
```typescript
interface Achievement {
  id: string;
  title: string;
  description: string;
  category: 'saving' | 'investing' | 'trading' | 'learning' | 'social' | 'web3';
  rarity: 'common' | 'rare' | 'epic' | 'legendary';
  reward: {
    type: 'rupees' | 'tokens' | 'experience' | 'unlock';
    amount: number;
  };
  criteria: AchievementCriteria;
}
```

**Sample Achievements:**

**Saving Category:**
- "First Deposit" - Make your first bank deposit (100 rupees)
- "Compound Master" - Earn 1,000 rupees in interest (500 rupees)
- "Patience Pays" - Keep money in fixed deposit for 30 days (1,000 rupees)

**Investing Category:**
- "Stock Market Debut" - Buy your first stock (200 rupees)
- "Diversified Portfolio" - Own stocks in 5 different sectors (1,000 rupees)
- "Diamond Hands" - Hold a stock for 90 days (500 rupees)

**Trading Category:**
- "Quick Profit" - Make 100 rupees in one trade (250 rupees)
- "Market Timer" - Buy low and sell high 10 times (1,000 rupees)
- "Risk Manager" - Use stop-loss orders successfully (300 rupees)

**Web3 Category:**
- "Crypto Curious" - Convert first rupees to tokens (100 tokens)
- "DeFi Explorer" - Participate in staking (200 tokens)
- "Blockchain Native" - Complete 10 Web3 transactions (500 tokens)

### Tutorial and Learning System

#### Progressive Tutorial Design
**Phase 1: Welcome & Basics (Levels 1-2)**
1. Character selection and customization
2. Basic movement and UI navigation
3. First rupee earning through simple tasks
4. Introduction to the game world and NPCs

**Phase 2: Banking Fundamentals (Levels 2-3)**
1. Visit the bank and meet the banker
2. Open first savings account
3. Make initial deposit and watch interest accrue
4. Learn about compound interest through visualization

**Phase 3: Investment Introduction (Levels 3-4)**
1. Visit the stock exchange
2. Research and buy first stock
3. Monitor price changes and portfolio value
4. Learn about risk and diversification

**Phase 4: Advanced Concepts (Levels 4-5)**
1. Create fixed deposits for higher returns
2. Try different investment strategies
3. Participate in market events and news reactions
4. Build a diversified portfolio

**Phase 5: Web3 Transition (Level 5+)**
1. Meet the Crypto Sage NPC
2. Learn about blockchain and decentralization
3. Convert rupees to tokens
4. Explore DeFi features and staking

#### Learning Reinforcement
**Spaced Repetition:**
- Key concepts revisited at increasing intervals
- Pop-up reminders about important principles
- Practice exercises to reinforce learning

**Visual Learning:**
- Charts and graphs for all financial concepts
- Animated explanations of complex topics
- Interactive calculators for compound interest, etc.

**Practical Application:**
- Every concept immediately practiced in-game
- Real scenarios with meaningful consequences
- Safe environment to make mistakes and learn

### Game Balance and Economy

#### Economic Balance
**Earning Rates:**
- **Basic Tasks**: 10-50 rupees per activity
- **Stock Trading**: Variable based on skill and luck
- **Interest Income**: 2-12% annually depending on product
- **Achievement Rewards**: 100-2,000 rupees

**Spending Opportunities:**
- **Stock Purchases**: 100-10,000 rupees per transaction
- **Fixed Deposits**: 500+ rupees minimum
- **Character Customization**: 50-500 rupees for cosmetics
- **Premium Features**: 1,000+ rupees for advanced tools

**Inflation Control:**
- Regular economic events that adjust prices
- Seasonal bonuses and challenges
- Progressive difficulty to maintain engagement
- Sink mechanisms to remove excess currency

#### Progression Pacing
**Time Investment:**
- **Daily Sessions**: 15-30 minutes for meaningful progress
- **Weekly Goals**: Achievable objectives for casual players
- **Monthly Challenges**: Longer-term goals for dedicated users
- **Seasonal Events**: Special content and rewards

**Skill Development:**
- **Beginner Phase**: 1-2 weeks to understand basics
- **Intermediate Phase**: 1-2 months to master core concepts
- **Advanced Phase**: 3-6 months for Web3 and complex strategies
- **Expert Phase**: Ongoing learning and community contribution

### Technical Implementation

#### Game State Management
```typescript
interface GameState {
  player: PlayerState;
  market: MarketState;
  social: SocialState;
  ui: UIState;
}

interface PlayerState {
  character: CharacterData;
  financial: FinancialData;
  progress: ProgressData;
  social: SocialData;
}
```

#### Real-time Updates
- **Market Prices**: Updated every 30 seconds
- **Interest Accrual**: Calculated and displayed in real-time
- **Social Interactions**: Instant messaging and notifications
- **Achievement Progress**: Live tracking and immediate rewards

#### Data Persistence
- **Local Storage**: UI preferences and temporary data
- **Database**: Player progress and financial data
- **Blockchain**: Web3 transactions and token balances
- **Hybrid Approach**: Seamless integration between systems

---

## Engagement Mechanics

### Daily Engagement Hooks

#### Morning Routine (9:00 AM)
- **Portfolio Update**: "Your investments earned X rupees overnight"
- **Market News**: "Today's trending stock: TECH (+5%)"
- **Daily Challenge**: "Complete today's financial quiz for bonus rupees"

#### Midday Check-in (12:00 PM)
- **Market Alert**: "Your ENERGY stock is up 10% - consider taking profits"
- **Social Update**: "Your friend just achieved 'Diversified Portfolio'"
- **Learning Tip**: "Did you know? Compound interest is the 8th wonder of the world"

#### Evening Wrap-up (6:00 PM)
- **Daily Summary**: "Today you earned X rupees and learned Y concepts"
- **Tomorrow Preview**: "Tomorrow's lesson: Understanding P/E ratios"
- **Streak Bonus**: "Day 7 of your learning streak - keep it up!"

### Retention Strategies

#### Habit Formation
- **Consistent Rewards**: Daily login bonuses
- **Progress Visualization**: Clear advancement through levels
- **Social Accountability**: Friend challenges and leaderboards
- **Fear of Missing Out**: Limited-time events and bonuses

#### Long-term Engagement
- **Evolving Content**: New stocks, events, and features
- **Seasonal Themes**: Holiday events and special challenges
- **Community Features**: User-generated content and discussions
- **Real-world Connections**: News integration and market events

This comprehensive gameplay system creates an engaging, educational experience that naturally progresses players from basic financial concepts to advanced Web3 understanding while maintaining high retention and user satisfaction.