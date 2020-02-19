---
title: "Typescript 实现状态模式"
date: 2020-02-11T14:07:25+08:00
description: "通过typescript实现状态模式"
tags: [ "前端","typescript"]
---
通过TypeScript实现状态模式
<!--more-->
# 状态模式

### 业务逻辑

![](http://o7ez1faxc.bkt.clouddn.com/2017-02-06-14863882728801.jpg)

### 初级版本

```javascript
const enum State{
    SOLD_OUT=0,//糖果售完
    NO_QUARTER=1,//没有硬币
    HAS_QUARTER=2,//已有硬币
    SOLD=3,//正在出糖果
    WINNER=4
}
class GumballMachine{
    public state:State;
    count:number;
    public constructor(count:number){
        this.state=State.SOLD
        this.count=count
        if(count>0){
            this.state=State.NO_QUARTER
        }
    }
    //插入硬币
    public insertQuarter():void{
        let currSte=this.state
        if(currSte==State.HAS_QUARTER){
            console.log('You can\'t insert another quarter')
        }else if(currSte==State.NO_QUARTER){
            this.state=State.HAS_QUARTER
            console.log('You inserted a quarter');
        }else if(currSte==State.SOLD_OUT){
            console.log('you can\'t insert a quarter,the machine is sold out');
        }else if(currSte==State.SOLD){
            console.log('Please wait,we\'re already giving you a gumball');
        }
        else if(currSte==State.WINNER){
            console.log('Please wait,we\'re already giving you a gumball');
        }
    }
    //退出硬币
    public ejectQuarter():void{
        let currSte=this.state
        if(currSte==State.HAS_QUARTER){
            console.log('Quarter returned');
            this.state=State.NO_QUARTER
        }
        else if(currSte==State.NO_QUARTER){
            console.log('you haven\'t inserted a quarter');
        }
        else if(currSte==State.SOLD){
            console.log('Sorry,you already turned the crank');
        }else if(currSte==State.WINNER){
            console.log('you can\'t eject,you haven\'t inserted a quarter yet')
        }
        else if(currSte==State.SOLD_OUT){
            console.log('you can\'t eject,you haven\'t inserted a quarter yet')
        }
    }
    //转动曲柄
    public turnCrank():void{
        if(this.state===State.SOLD){
            console.log('Turning twice doesn\'t get you another gumball!');
        }
        else if(this.state===State.NO_QUARTER){
            console.log("you turned but there's no quarter");
        }
        else if(this.state===State.SOLD_OUT){
            console.log("You turned,but there are no gumballs")
        }
        else if(this.state===State.HAS_QUARTER){
            console.log("you turned...")
            if(Math.random()>0.9){
                this.state=State.WINNER
            }else{

                this.state=State.SOLD
            }
            this.dispense();
        }
    }
    //弹出糖果
    public dispense():void{
        if(this.state==State.SOLD){
            console.log("A gumball comes rolling out the slot");
            this.count-=1
            if(this.count==0){
                console.log("Oops,out of gumballs!");
                this.state=State.SOLD_OUT
            }else{
                this.state=State.NO_QUARTER
            }
        }else if(this.state==State.WINNER){
            console.log("two gumball comes rolling out the slot")
            this.count-=2
            if(this.count==0){
                console.log("Oops,out of gumballs!");
                this.state=State.SOLD_OUT
            }else{
                this.state=State.NO_QUARTER
            }
        }else if(this.state==State.NO_QUARTER){
            console.log("You need to pay first");
        }else if(this.state==State.SOLD_OUT){
            console.log("No gumball dispensed");
        }else if(this.state==State.HAS_QUARTER){
            console.log("No gumball dispensed");
        }

    }
    public toString():any{
          if(this.state==State.SOLD){
              return "current State:SOLD"+","+"count:"+this.count
        }else if(this.state==State.NO_QUARTER){
            return "current NO_QUARTER"+","+"count:"+this.count
        }else if(this.state==State.SOLD_OUT){
            return "current SOLD_OUT"+","+"count:"+this.count
        }else if(this.state==State.HAS_QUARTER){
            return "current HAS_QUARTER"+","+"count:"+this.count
        }
    }
}
//测试代码
let gumballMachine:GumballMachine
gumballMachine=new GumballMachine(5)
console.log(gumballMachine.toString());
gumballMachine.insertQuarter();
gumballMachine.turnCrank();
console.log(gumballMachine.toString());

gumballMachine.insertQuarter()
gumballMachine.ejectQuarter()
gumballMachine.turnCrank()
console.log(gumballMachine.toString());

gumballMachine.insertQuarter()
gumballMachine.turnCrank()
gumballMachine.insertQuarter()
gumballMachine.turnCrank()
gumballMachine.ejectQuarter()
console.log(gumballMachine.toString());
console.log('------------------------');

gumballMachine.insertQuarter()
gumballMachine.insertQuarter()
gumballMachine.turnCrank()
gumballMachine.insertQuarter()
gumballMachine.turnCrank()
gumballMachine.insertQuarter()
gumballMachine.turnCrank()
console.log(gumballMachine.toString());

```

### 改良版本

![](http://o7ez1faxc.bkt.clouddn.com/2017-02-06-14863884718479.jpg)

```typescript
/*
const enum State {
    
    SOLD_OUT = 0,
    NO_QUARTER = 1,
    HAS_QUARTER = 2,
    SOLD = 3,
    WINNER = 4
}
*/
interface State {

    state: State;
    count: number;
    insertQuarter(): void;
    ejectQuarter():void;
    turnCrank():void;
    dispense():void;
}
class SoldOutState implements State{
    state:State;
    count:number;
    gumballMachine:GumballMachine
    constructor(gumballMachine:GumballMachine){
        this.gumballMachine=gumballMachine
    }
    public insertQuarter():void{
        console.log("You can't insert a quarter,the machine is sold out");
    }
    public ejectQuarter():void{
        console.log("You can't eject,you haven't inserted a quater yet");
    }
    public turnCrank():void{
        console.log("you turned,but there are no gumballs");
    }
    public dispense():void{
        console.log("No gumball dispensed");
    }
}

class NoQuarterState implements State {
    state: State
    count: number;
    gumballMachine: GumballMachine
    constructor(gumballMachine: GumballMachine) {
        this.gumballMachine = gumballMachine
    }
    public insertQuarter(): void {
        console.log("You inserted a quarter");
        this.gumballMachine.setState(this.gumballMachine.getHasQuarterState());
    }
    public ejectQuarter() :void{
        console.log('you haven\'t inserted a quarter');
    }
    public turnCrank():void {
        console.log("You turned,but there's no quarter");
    }
    public dispense():void {
        console.log("You need to pay first");
    }
}
class SoldState implements State {
    state: State
    count: number;
    gumballMachine: GumballMachine
    constructor(gumballMachine: GumballMachine) {
        this.gumballMachine=gumballMachine

    }
    public insertQuarter(): void {
        console.log("Please wait,we're already giving you a gumball");
    }
    public ejectQuarter():void {
        console.log("Sorry,you already turned the crank");
    }
    public turnCrank():void {
        console.log("Turning twice doesn't get you another gumball!");
    }
    public dispense():void {
        this.gumballMachine.releaseBall();
        if(this.gumballMachine.getCount()>0){
            this.gumballMachine.setState(this.gumballMachine.getNoQuarterState());
        }else{
            console.log("oops,out of gumballs!");
            this.gumballMachine.setState(this.gumballMachine.getSoldOutState());

        }
    }
}
class HasQuarterState implements State {
    state: State
    count: number;
    gumballMachine: GumballMachine
    constructor(gumballMachine: GumballMachine) {
        this.gumballMachine=gumballMachine
        console.log("has quarter")

    }
    public insertQuarter(): void {
        console.log("you can't insert another quarter");
    }
    public ejectQuarter():void {
        console.log("Quarter returned");
        this.gumballMachine.setState(this.gumballMachine.getNoQuarterState())
    }
    public turnCrank():void {
        console.log("You turned...");
        this.gumballMachine.setState(this.gumballMachine.getSoldState())
    }
    public dispense():void {
        this.gumballMachine.releaseBall();
        if(this.gumballMachine.getCount()>0){
            this.gumballMachine.setState(this.gumballMachine.getNoQuarterState());
        }else{
            console.log("oops,out of gumballs!");
            this.gumballMachine.setState(this.gumballMachine.getSoldOutState());

        }
    }
}
class GumballMachine {
    soldOutState: State;
    noQuarterState: State
    hasQuarterState: State
    soldState: State
    state: State = this.soldOutState
    count: number = 0
    constructor(numberGumballs: number) {
        this.noQuarterState = new NoQuarterState(this);
        this.soldOutState = new SoldOutState(this);
        this.hasQuarterState=new HasQuarterState(this);
        this.soldState=new SoldState(this);
        this.count=numberGumballs
        if (numberGumballs > 0) {
            this.state = this.noQuarterState
        }
    }
    public insertQuarter(): void {
        this.state.insertQuarter();
    }
    public turnCrank(): void {
        this.state.turnCrank();
        this.state.dispense();
    }
    /**
     * setState
     */
    public setState(state: State):void {
        this.state = state
    }
    public releaseBall(): void {
        console.log("A gumball comes rolling out the slot...");
        if (this.count != 0) {
            this.count = this.count - 1
        }
    }

    public getHasQuarterState(): State {
        return this.hasQuarterState
    }
    public getCount():number{
        return this.count
    }
    public getNoQuarterState():State{
        return this.noQuarterState
    }
    public getSoldOutState():State{
        return this.soldOutState
    }
    public getSoldState():State{
        return this.soldState
    }
    public toString():string{
        return this.state+':'+this.count
    }
}
let gumballMachine
gumballMachine=new GumballMachine(5)
console.log(gumballMachine.toString());

gumballMachine.insertQuarter();
gumballMachine.turnCrank();
console.log(gumballMachine.toString());
console.log('-----------------------');
gumballMachine.insertQuarter();
gumballMachine.insertQuarter();
gumballMachine.turnCrank();
gumballMachine.insertQuarter();
gumballMachine.turnCrank();

console.log(gumballMachine.toString());

```

