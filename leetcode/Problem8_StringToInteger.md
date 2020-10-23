# 题目链接

https://leetcode-cn.com/problems/string-to-integer-atoi/

# 解法1
思路很简答， 就是遍历字符串。注意，累加数值时要及时退出循环，防止溢出。

执行用时超过50%的java提交记录。

```java
    public int myAtoi(String s) {
        char[] chars = s.toCharArray();

        int i = 0;
        while (i < chars.length && Character.isSpaceChar(chars[i])) {
            i++;
        }

        if (i == chars.length
            || (chars[i] != '+' && chars[i] != '-' && !Character.isDigit(chars[i]))) {
            return 0;
        }

        boolean negative = false;
        if (chars[i] == '+' || chars[i] == '-') {
            negative = chars[i] == '-';
            i++;
        }

        long ret = 0;
        while (i < chars.length && Character.isDigit(chars[i])) {
            ret = ret * 10 + (chars[i]-'0');
            if (ret > Integer.MAX_VALUE) {
                break;
            }
            i++;
        }
        ret = ret * (negative ? -1 : 1);

        if (ret > Integer.MAX_VALUE) {
            return Integer.MAX_VALUE;
        } else if (ret < Integer.MIN_VALUE) {
            return Integer.MIN_VALUE;
        } else {
            return (int)ret;
        }
    }
```

# 解法2

借鉴state pattern重写了下，似乎是高级了一点。。。

然而，执行用时还增加了，只超过了29%的Java提交。

```java
    public int myAtoiV2(String s) {
        StatedString ss = new StatedString(s);
        return ss.atoi();
    }

    private interface State {
        void transfer(char ch);
    }

    private class StatedString {
        private String s;

        private State spaceState;
        private State signState;
        private State numberState;
        private State endState;

        private State state;

        StatedString(String s) {
            this.s = s;
            this.spaceState = new SpaceState(this);
            this.signState = new SignState(this);
            this.numberState = new NumberState(this);
            this.endState = new EndState(this);
            this.state = spaceState;
        }

        public State getSpaceState() {
            return spaceState;
        }

        public State getSignState() {
            return signState;
        }

        public State getNumberState() {
            return numberState;
        }

        public State getEndState() {
            return endState;
        }

        public void setState(State state) {
            this.state = state;
        }

        public int atoi() {
            char[] chars = s.toCharArray();

            boolean negative = false;
            long ret = 0;
            for (char ch : chars) {
                state.transfer(ch);
                if (state.equals(signState)) {
                    negative = ch == '-' ? true : false;
                } else if (state.equals(numberState)) {
                    ret = ret * 10 + (ch - '0');
                    if (ret > Integer.MAX_VALUE) {
                        break;
                    }
                } else if (state.equals(endState)) {
                    break;
                }
            }
            ret = ret * (negative ? -1 : 1);

            if (ret > Integer.MAX_VALUE) {
                return Integer.MAX_VALUE;
            } else if (ret < Integer.MIN_VALUE) {
                return Integer.MIN_VALUE;
            } else {
                return (int)ret;
            }
        }
    }

    private class SpaceState implements State {
        private StatedString ss;

        SpaceState(StatedString ss) {
            this.ss = ss;
        }

        @Override
        public void transfer(char ch) {
            if (Character.isSpaceChar(ch)) {
                ss.setState(ss.getSpaceState());
            } else if (ch == '+' || ch == '-') {
                ss.setState(ss.getSignState());
            } else if (Character.isDigit(ch)) {
                ss.setState(ss.getNumberState());
            } else {
                ss.setState(ss.getEndState());
            }
        }
    }

    private class SignState implements State {
        private StatedString ss;

        SignState(StatedString ss) {
            this.ss = ss;
        }

        @Override
        public void transfer(char ch) {
            if (Character.isDigit(ch)) {
                ss.setState(ss.getNumberState());
            } else {
                ss.setState(ss.getEndState());
            }
        }
    }

    private class NumberState implements State {
        private StatedString ss;

        NumberState(StatedString ss) {
            this.ss = ss;
        }

        @Override
        public void transfer(char ch) {
            if (Character.isDigit(ch)) {
                ss.setState(ss.getNumberState());
            } else {
                ss.setState(ss.getEndState());
            }
        }
    }

    private class EndState implements State {
        private StatedString ss;

        EndState(StatedString ss) {
            this.ss = ss;
        }

        @Override
        public void transfer(char ch) {
            ss.setState(ss.getEndState());
        }
    }
```

