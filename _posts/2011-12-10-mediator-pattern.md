---
layout: post
title: Mediator Pattern
date: '2011-12-10T08:42:35+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118512493016/mediator-pattern
---
여러 객체들이 서로 협력을 할 필요가 있을 때, Mediator Pattern이 유용하게 사용할 수 있다.

만약 Mediator Pattern을 사용하지 않고 여러 객체들이 서로 협력을 하기 위해서는 서로간의 연결이 필요하게 되고 이 연결은 복잡해지기 쉽다.

Mediator Pattern을 이용한다면, 복잡한 연결을 Mediator라는 중재자를 통해서 여러 객체가 협력하도록 할 수있다. 즉 1:N 연결이 되므로 협력이 더욱 쉬워진다.
Mediator Pattern에서 객체 간의 협력과 관련된 부분은 Mediator에 한정이 되어 있게 된다. 여기에서 협력과 관련된 부분은 프로그램에 의존적인 부분이며, 재이용성이 낮아진다.

하지만, Mediator가 관리하는 Colleague들 즉 서로 협력하기 위한 객체들은 재이용성이 높아져 다른 프로그램에서도 재사용할 수가 있게 된다.
일반적인 구조
<a href="http://i1.wp.com/appchemist.net/wp-content/uploads/2011/12/mediator.png"><img src="http://i1.wp.com/appchemist.net/wp-content/uploads/2011/12/mediator.png?resize=409%2C212" alt="mediator" class="aligncenter size-full wp-image-642" data-recalc-dims="1"/></a>

예제 소스 (출처 : Java 언어로 배우는 디자인 패턴 입문. 영진닷컴)


```java
public class MediatorPattern {
    static public void main(String args[]) {
        new LoginFrame("Mediator Sample");
    }
}

interface Mediator {
    public void createColleagues();
    public void colleagueChanged();
}

interface Colleague {
    public void setMediator(Mediator mediator);
    public void setColleagueEnabled(boolean enabled);
}

class ColleagueButton extends Button implements Colleague {
    private Mediator mediator;
    public ColleagueButton(String caption) {
        super(caption);
    }
    public void setMediator(Mediator mediator) {
        this.mediator = mediator;
    }
    public void setColleagueEnabled(boolean enabled) {
        setEnabled(enabled);
    }
}

class ColleagueTextField extends TextField implements TextListener, Colleague {
    private Mediator mediator;
    public ColleagueTextField(String text, int columns) {
        super(text, columns);
    }
   
    public void setMediator(Mediator mediator) {
        this.mediator = mediator;
    }
   
    public void setColleagueEnabled(boolean enabled) {
        setEnabled(enabled);
        setBackground(enabled ? Color.white : Color.lightGray);
    }
   
    public void textValueChanged(TextEvent e) {
        mediator.colleagueChanged();
    }
}

class ColleagueCheckbox extends Checkbox implements ItemListener, Colleague {
    private Mediator mediator;
    public ColleagueCheckbox(String caption, CheckboxGroup group, boolean state) {
        super(caption, group, state);
    }
    public void setMediator(Mediator mediator) {
        this.mediator = mediator;
    }
    public void setColleagueEnabled(boolean enabled) {
        setEnabled(enabled);
    }
    public void itemStateChanged(ItemEvent e) {
        mediator.colleagueChanged();
    }
}

class LoginFrame extends Frame implements ActionListener, Mediator {
    private ColleagueCheckbox checkGuest;
    private ColleagueCheckbox checkLogin;
    private ColleagueTextField textUser;
    private ColleagueTextField textPass;
    private ColleagueButton buttonOk;
    private ColleagueButton buttonCancel;
   
    public LoginFrame(String title) {
        super(title);
        setBackground(Color.lightGray);
        setLayout(new GridLayout(4, 2));
        createColleagues();
       
        add(checkGuest);
        add(checkLogin);
        add(new Label("Username : "));
        add(textUser);
        add(new Label("Password : "));
        add(textPass);
        add(buttonOk);
        add(buttonCancel);
        colleagueChanged();
        pack();
        show();
    }
   
    public void createColleagues() {
        CheckboxGroup g = new CheckboxGroup();
        checkGuest = new ColleagueCheckbox("Guest", g, true);
        checkLogin = new ColleagueCheckbox("Login", g, false);
        textUser = new ColleagueTextField(" ", 10);
        textPass = new ColleagueTextField(" ", 10);
        textPass.setEchoChar('*');
        buttonOk = new ColleagueButton("OK");
        buttonCancel = new ColleagueButton("Cancel");
       
        checkGuest.setMediator(this);
        checkLogin.setMediator(this);
        textUser.setMediator(this);
        textPass.setMediator(this);
        buttonOk.setMediator(this);
        buttonCancel.setMediator(this);
       
        checkGuest.addItemListener(checkGuest);
        checkLogin.addItemListener(checkLogin);
        textUser.addTextListener(textUser);
        textPass.addTextListener(textPass);
        buttonOk.addActionListener(this);
        buttonCancel.addActionListener(this);
    }
   
    public void colleagueChanged() {
        if(checkGuest.getState()) {
            textUser.setColleagueEnabled(false);
            textPass.setColleagueEnabled(false);
            buttonOk.setColleagueEnabled(true);
        } else {
            textUser.setColleagueEnabled(true);
            userpassChanged();
        }
    }
   
    private void userpassChanged() {
        if(textUser.getText().length() &gt; 0) {
            textPass.setColleagueEnabled(true);
            if(textPass.getText().length() &gt; 0) {
                buttonOk.setColleagueEnabled(true);
            } else {
                buttonOk.setColleagueEnabled(false);
            }
        } else {
            textPass.setColleagueEnabled(false);
            buttonOk.setColleagueEnabled(false);
        }
    }
    public void actionPerformed(ActionEvent e) {
        System.out.println(e.toString());
        System.exit(0);
    }
}
```
