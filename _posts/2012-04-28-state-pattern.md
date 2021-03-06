---
layout: post
title: State Pattern
date: '2012-04-28T12:56:12+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118512614056/state-pattern
---
OOP에서 프로그램 할 대상을 ‘클래스’로 표현한다. 일반적으로 클래스는 구체적인 사물 또는 존재하지 않는 것을 표현한다. 여기에서 State Pattern은 ‘상태’를 클래스로 표현한 패턴이다.
1. divide and conquer : 상태 클래스로 표현하여 복잡한 프로그램을 분할한다.

State Pattern은 한 가지 상태에 대해서 처리 할 내용을 한 클래스에 모아둘 수 있다는 장점이 생긴다.
이 장점은 C로 상태 전이도를 표현해 본 사람이라면 이 패턴이 매우 매력적으로 느껴질 것 같다. 상태 전이도를 표현하기 위해서는 수 많은 if문을 이용해서 다양한 경우를 분리해 내용을 처리해야 한다. 이러한 코드들도 한 메소드에 모이기 쉽다. 나중에 다시 코드를 들여다 본다면 그 내용을 알아보기 힘들 것이다.
상태 전이도를 State Pattern으로 표현한다면, 한 가지 상태에서 처리해야 할 내용을 한 클래스에 모아둘 수 있다. 결국에 if문이 줄어들게 되고 복잡한 상태를 손쉽게 알아보며 코딩을 할 수 있다는 장점이 있다.
2.상태 의존 처리

Context는 State 인터페이스를 가지고 있다. 즉, 인터페이스는 상황에 따라 다른 상태 클래스를 가지게 되고 그 상태에 따른 처리를 할 수 있다.
3.자기 모순에 빠지지 않는다.

위에서 상태 전이도에 대해서 이야기를 했다. 여기에서 많은 상태가 존재한다면 다양한 상태에 대해서 처리를 모두 염두에 두고 프로그래밍을 해야 하므로 모순에 빠지기 쉽다.

하지만 패턴을 이용한다면 한 상태에 집중하여 처리가 가능하므로 난이도가 낮아 지게 된다.
4. 새로운 상태를 추가하기가 쉽다.

모든 상태를 한 곳에 다 모아두었다면, 상태를 추가하거나 삭제하기 위해서는 그 코드에 대해서 정확히 이해를 해야하지만 실수하기도 쉽다.

하지만 State 패턴은 상태를 추가하고 삭제하는 것은 State 인터페이스를 구현해 주면 된다.

단, State 인터페이스를 구현한 상태 클래스에서 상태를 변경한다면 상태 클래스간 의존성이 높아지게 돼 추가, 삭제하는 것이 힘들어 지게 된다.
일반적인 구조
<a href="http://i2.wp.com/appchemist.net/wp-content/uploads/2012/04/state_pattern.png"><img src="http://i2.wp.com/appchemist.net/wp-content/uploads/2012/04/state_pattern.png?resize=470%2C170" alt="state_pattern" class="aligncenter size-full wp-image-634" data-recalc-dims="1"/></a>
예제 소스(출처 : Java 언어로 배우는 디자인 패턴 입문. 영진닷컴)


```java
import java.awt.BorderLayout;
import java.awt.Button;
import java.awt.Color;
import java.awt.Frame;
import java.awt.Panel;
import java.awt.TextArea;
import java.awt.TextField;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;


public class StatePattern {
    public static void main(String[] args) {
        SafeFrame frame = new SafeFrame("State Sample");
        while (true) {
            for (int hour = 0; hour &lt; 24; hour++) {
                frame.setClock(hour);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                }
            }
        }
    }
}

interface State {
    public void doClock(Context context, int hour);
    public void doUse(Context context);
    public void doAlarm(Context context);
    public void doPhone(Context context);
}

class DayState implements State {
    private static DayState singleton = new DayState();
    private DayState() {
    }
   
    public static State getInstance() {
        return singleton;
    }
   
    public void doClock(Context context, int hour) {
        if (hour &lt; 9 || 17 &lt;= hour) {
            context.changeState(NightState.getInstance());
        }
    }
   
    public void doUse(Context context) {
        context.recordLog("금고사용(주간)");
    }
   
    public void doAlarm(Context context) {
        context.callSecurityCenter("비상벨(주간)");
    }
   
    public void doPhone(Context context) {
        context.callSecurityCenter("인반통화(주간)");
    }
   
    public String toString() {
        return "(주간)";
    }
}

class NightState implements State {
    private static NightState singleton = new NightState();
   
    private NightState() {
    }
   
    public static State getInstance() {
        return singleton;
    }
   
    public void doClock(Context context, int hour) {
        if (9 &lt;= hour &amp;&amp; hour &lt; 17) {
            context.changeState(DayState.getInstance());
        }
    }
   
    public void doUse(Context context) {
        context.callSecurityCenter("비상 : 야간금고 사용!");
    }
   
    public void doAlarm(Context context) {
        context.callSecurityCenter("비상벨(야간)");
    }
   
    public void doPhone(Context context) {
        context.recordLog("야간통화 녹음");
    }
   
    public String toString() {
        return "[야간]";
    }
}

interface Context {
    public void setClock(int hour);
    public void changeState(State state);
    public void callSecurityCenter(String msg);
    public void recordLog(String msg);
}

class SafeFrame extends Frame implements ActionListener, Context {
    private TextField textClock = new TextField(60);
    private TextArea textScreen = new TextArea(10, 60);
    private Button buttonUse = new Button("금고사용");
    private Button buttonAlarm = new Button("비상벨");
    private Button buttonPhone = new Button("일반통화");
    private Button buttonExit = new Button("종료");
   
    private State state = DayState.getInstance();
   
    public SafeFrame(String title) {
        super(title);
        setBackground(Color.lightGray);
        setLayout(new BorderLayout());
       
        add(textClock, BorderLayout.NORTH);
        textClock.setEditable(false);
       
        add(textScreen, BorderLayout.CENTER);
        textScreen.setEditable(false);
       
        Panel panel = new Panel();
        panel.add(buttonUse);
        panel.add(buttonAlarm);
        panel.add(buttonPhone);
        panel.add(buttonExit);
       
        add(panel, BorderLayout.SOUTH);
       
        pack();
        show();
       
        buttonUse.addActionListener(this);
        buttonAlarm.addActionListener(this);
        buttonPhone.addActionListener(this);
        buttonExit.addActionListener(this);
    }
   
    public void actionPerformed(ActionEvent e) {
        System.out.println(e.toString());
        if (e.getSource() == buttonUse) {
            state.doUse(this);
        } else if (e.getSource() == buttonAlarm) {
            state.doAlarm(this);
        } else if (e.getSource() == buttonPhone) {
            state.doPhone(this);
        } else if (e.getSource() == buttonExit) {
            System.exit(0);
        } else {
            System.out.println("?");
        }
    }
   
    public void setClock(int hour) {
        String clockstring = "현재 시간은";
        if (hour &lt; 10) {
            clockstring += "0" + hour + ":00";
        } else {
            clockstring += hour + ":00";
        }
        System.out.println(clockstring);
        textClock.setText(clockstring);
        state.doClock(this, hour);
    }
   
    public void changeState(State state) {
        System.out.println(this.state + "에서" + state + "로 상태가 변화했습니다.");
        this.state = state;
    }
   
    public void callSecurityCenter(String msg) {
        textScreen.append("call! " + msg + "n");
    }
   
    public void recordLog(String msg) {
        textScreen.append("record ..." + msg + "n");
    }
}
```
