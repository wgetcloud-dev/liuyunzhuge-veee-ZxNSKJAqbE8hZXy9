
状态机（State Machine）是一种设计模式，用于描述对象在不同状态之间的转换和行为。状态机可以帮助开发者管理复杂的状态逻辑，使得系统在不同状态下的行为更易于理解和维护。以下是关于状态机设计模式的详细介绍。


### 1\. 状态机的基本概念


* **状态**：表示对象在某一时刻的情况或条件。例如，订单的状态可以是“新建”、“处理中”、“已完成”。
* **事件**：导致状态变化的触发器，例如用户操作、时间到达或其他外部输入。
* **状态转移**：从一个状态到另一个状态的过程，通常由事件驱动。
* **上下文**：持有状态机当前状态及相关数据的对象。


### 2\. 状态机的组成部分


状态机通常由以下几个组成部分构成：


* **状态（States）**：定义了系统可能处于的所有状态。
* **事件（Events）**：触发状态变迁的事件。
* **转移（Transitions）**：描述状态之间的转换规则，通常与特定事件关联。
* **行为（Actions）**：在状态进入、退出或转移时执行的操作。


### 3\. 状态机的类型


状态机可以分为两种主要类型：


* **有限状态机（Finite State Machine, FSM）**：状态数量有限，适用于大多数场景。
* **层次状态机（Hierarchical State Machine）**：允许状态嵌套，可以更好地组织复杂的状态逻辑。


### 4\. 状态机的优点


* **清晰性**：将状态和行为明确分开，使代码更加易读和可维护。
* **灵活性**：便于修改和扩展状态逻辑，例如添加新的状态或事件。
* **可测试性**：每个状态和转移都可以独立测试，提高了系统的可靠性。


### 5\. 状态机的实现


状态机可以通过多种方式实现，常见的方法包括：


* **使用条件语句（if\-else 或 switch\-case）**：简单的状态机可以用条件语句直接实现，但随着状态和事件的增加，代码会变得复杂且难以维护。
* **状态模式（State Pattern）**：一种面向对象的设计模式，通过创建状态类来封装状态相关的行为，实现动态状态切换。
* **状态机框架**：使用现有的状态机库或框架（如 `Stateless4j`、`Spring State Machine` 等）来简化状态机的创建和管理。


### 6\. 状态模式示例


下面是一个使用状态模式实现简单状态机的示例，展示如何管理订单的状态。



```
// 定义状态接口
interface OrderState {
    void handle(OrderContext context);
}

// 具体状态实现
class NewOrderState implements OrderState {
    public void handle(OrderContext context) {
        System.out.println("Handling new order.");
        context.setState(new ProcessingOrderState());
    }
}

class ProcessingOrderState implements OrderState {
    public void handle(OrderContext context) {
        System.out.println("Processing order.");
        context.setState(new CompletedOrderState());
    }
}

class CompletedOrderState implements OrderState {
    public void handle(OrderContext context) {
        System.out.println("Order completed.");
    }
}

// 上下文类
class OrderContext {
    private OrderState state;

    public OrderContext() {
        this.state = new NewOrderState(); // 初始状态
    }

    public void setState(OrderState state) {
        this.state = state;
    }

    public void request() {
        state.handle(this);
    }
}

// 客户端代码
public class StatePatternDemo {
    public static void main(String[] args) {
        OrderContext order = new OrderContext();
        order.request(); // 处理新订单
        order.request(); // 处理中的订单
        order.request(); // 订单已完成
    }
}

```

### 7\. 总结


状态机设计模式是管理复杂状态和行为的一种有效方法。通过将状态、事件和转移逻辑清晰地分开，状态机使得程序的结构更加清晰，易于理解和维护。无论是在游戏开发、工作流管理还是其他需要状态管理的应用中，状态机都是一种非常有用的工具。使用现有的状态机框架可以进一步简化开发过程，提高效率。


# 轻量级状态机stateless4j的使用


* 项目地址：[https://github.com/stateless4j/stateless4j](https://github.com)
* 可以把当前状态和状态字段持久化的数据库里，实例中存储到文件里
resources/stateless.json



```
[
  {
    "state": "待付款",
    "trigger": "用户付款",
    "destinationState": "待发货"
  },
  {
    "state": "待发货",
    "trigger": "发货",
    "destinationState": "待收货"
  },
  {
    "state": "待收货",
    "trigger": "确认收货",
    "destinationState": "已完成"
  },
  {
    "state": "已完成",
    "trigger": "评价",
    "destinationState": "已评价"
  },
  {
    "state": "已评价",
    "trigger": "删除",
    "destinationState": "已删除"
  }
]


```

测试用例



```
/**
	 * 状态从file中读取
	 */
	@Test
	public void stateFromDb() throws IOException {
		String content = FileUtils.readToEnd(
				StateMachineExample.class.getClassLoader().getResourceAsStream("stateless.json"),
				Charset.defaultCharset());
		StateMachineConfig stateMachineConfig = new StateMachineConfig<>();

		List> map = objectMapper.readValue(content, new TypeReference>>() {
		});
		map.forEach(o -> {
			String state = o.get("state");
			String trigger = o.get("trigger");
			String destinationState = o.get("destinationState");
			stateMachineConfig.configure(state).permit(trigger, destinationState);
		});
		StateMachine stateMachine = new StateMachine<>("待收货", stateMachineConfig);
		log.info("当前状态：{}", stateMachine.getState());
		stateMachine.fire("确认收货");
		log.info("当前状态：{}", stateMachine.getState());

	}

```

结果
![](https://img2024.cnblogs.com/blog/118538/202412/118538-20241226113805419-1990276773.png)
如果你的state和trigger不一致，比如状态是`待收获`，你的动作是`评价`，那状态机就会报错，状态不能正确更新
![](https://img2024.cnblogs.com/blog/118538/202412/118538-20241226113943869-1355964045.png)


 本博客参考[veee加速器官网](https://youhaochi.com)。转载请注明出处！
