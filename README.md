# CommBank Program - Full Submission

## Repositories
- **Backend:** https://github.com/vrk24/CommBank-Server
- **Frontend:** https://github.com/vrk24/CommBank-Web

---

# Task 1 - Backend

## Changes Made
- Forked and set up .NET server
- Connected to MongoDB Atlas
- Seeded the database with 5 collections
- Added optional `Icon` field to Goal model

### Goal.cs - Added Icon field
```csharp
using MongoDB.Bson;
using MongoDB.Bson.Serialization.Attributes;

namespace CommBank.Models;

public class Goal
{
    [BsonId]
    [BsonRepresentation(BsonType.ObjectId)]
    public string? Id { get; set; }

    public string? Name { get; set; }

    public UInt64 TargetAmount { get; set; } = 0;

    public DateTime TargetDate { get; set; }

    public double Balance { get; set; } = 0.00;

    public DateTime Created { get; set; } = DateTime.Now;

    [BsonRepresentation(BsonType.ObjectId)]
    public List<string>? TransactionIds { get; set; }

    [BsonRepresentation(BsonType.ObjectId)]
    public List<string>? TagIds { get; set; }

    [BsonRepresentation(BsonType.ObjectId)]
    public string? UserId { get; set; }

    public string? Icon { get; set; }
}
```

---

# Task 2 - Frontend

## Changes Made
- Forked and set up React web app
- Updated Goal model with optional `icon` field
- Implemented emoji picker
- Users can add and change icons on goals

### types.ts - Updated Goal interface
```typescript
export interface Goal {
  id: string
  name: string
  targetAmount: number
  balance: number
  targetDate: Date
  created: Date
  accountId: string
  transactionIds: string[]
  tagIds: string[]
  icon?: string
}
```

### GoalManager.tsx - Emoji picker implementation
```typescript
const [icon, setIcon] = useState<string | null>(null)
const [showEmojiPicker, setShowEmojiPicker] = useState(false)

const pickEmojiOnClick = (emoji: BaseEmoji, event: React.MouseEvent) => {
    const nextIcon = emoji.native
    setIcon(nextIcon)
    setShowEmojiPicker(false)
    const updatedGoal: Goal = { ...props.goal, icon: nextIcon }
    dispatch(updateGoalRedux(updatedGoal))
    updateGoalApi(props.goal.id, updatedGoal)
}

const toggleEmojiPicker = (e: React.MouseEvent) => {
    setShowEmojiPicker(!showEmojiPicker)
}
```

---

# Task 3 - Full Stack (Emoji Persistence)

## Changes Made
- Updated `API_ROOT` in `lib.ts` to point to local server
- Implemented `pickEmojiOnClick` to persist emoji via PUT request
- Emoji icons now persist after page refresh

### lib.ts - Updated API_ROOT
```typescript
export const API_ROOT = 'http://localhost:5203'
```

### lib.ts - PUT request
```typescript
export async function updateGoal(goalId: string, updatedGoal: Goal): Promise<boolean> {
  try {
    await axios.put(`${API_ROOT}/api/Goal/${goalId}`, updatedGoal)
    return true
  } catch (error: any) {
    return false
  }
}
```

### GoalManager.tsx - pickEmojiOnClick handler
```typescript
const pickEmojiOnClick = (emoji: BaseEmoji, event: React.MouseEvent) => {
    const nextIcon = emoji.native
    setIcon(nextIcon)
    setShowEmojiPicker(false)
    const updatedGoal: Goal = { ...props.goal, icon: nextIcon }
    dispatch(updateGoalRedux(updatedGoal))
    updateGoalApi(props.goal.id, updatedGoal)
}
```

---

# Task 4 - Tests

## Changes Made
- Implemented `GetForUser` test in `GoalControllerTests.cs`
- All 11 tests passing

### GoalControllerTests.cs - GetForUser test
```csharp
[Fact]
public async void GetForUser()
{
    // Arrange
    var goals = collections.GetGoals();
    var users = collections.GetUsers();
    IGoalsService goalsService = new FakeGoalsService(goals, goals[0]);
    IUsersService usersService = new FakeUsersService(users, users[0]);
    GoalController controller = new(goalsService, usersService);

    // Act
    var httpContext = new Microsoft.AspNetCore.Http.DefaultHttpContext();
    controller.ControllerContext.HttpContext = httpContext;
    var result = await controller.GetForUser(users[0].Id!);

    // Assert
    Assert.NotNull(result);
    var index = 0;
    foreach (Goal goal in result!)
    {
        Assert.IsAssignableFrom<Goal>(goal);
        Assert.Equal(goals[index].Id, goal.Id);
        Assert.Equal(goals[index].Name, goal.Name);
        index++;
    }
}
```