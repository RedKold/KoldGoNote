```csharp
  SlicedEvents.Add("CardPlayed", new SlicedEvent<(Entity, Card, Box<Entity>, Context)>());
```


```csharp
// in Controller/CombatManager.cs
public static void Play(Entity instigator, Card card, Box<Entity>? targets)
		{
			card.Commands[0]?.ForEach(cmd =>{
				SlicedEvents.Trigger("CardPlayed", cmd, (instigator, card, targets, _cnt));
			});
		}
```

打出卡牌事件 `CardPlayed` 是一个接收 `Entity instigator, Card card, Box<Entity>? targets` 的事件。

我们在打出打牌的地方 `Trigger` 这个事件即可。

对于卡牌打出，我们在检测到 `Drag` 到打牌区，就打出这张牌（实际是调用打牌函数）



```csharp
public void ForEach(Action<T, uint> action)
{
	for (int i = 0; i < _count; i++)
	{
		if (_slots[i] is not null)
			action(_slots[i], (uint)i);
	}
}
```

`Action<T, uint> action`, `T, for example Card`



```csharp
public class Player : Entity
    {
#nullable enable
        public readonly string Name;
        public Slots<Card> Inventory { get; private set; }
        public Slots<Card> Situation { get; private set; }
        public Card? ArmorSlot;
        private Dictionary<Material, uint> _material_bag = new(23);

        internal Player(string name, double maxHealth, ITags tags, uint ivt = 36, uint sta = 3) : base(maxHealth, tags)
        {
            Name = name;
            Inventory = new(ivt);
            Situation = new(sta);
        }

        internal Player(string name, double maxHealth, uint ivt = 36, uint sta = 3) : base(maxHealth)
        {
            Name = name;
            Inventory = new(ivt);
            Situation = new(sta);
        }

        internal void Play(uint index, Box<Entity>? targets)
        {
            Card card = Inventory[index];
            Events.Trigger("CardDurabilityDamaged", (card, (uint)1));
            CombatManager.Play(this, card, targets);
        }
#nullable disable
    }
```

- `Player` 