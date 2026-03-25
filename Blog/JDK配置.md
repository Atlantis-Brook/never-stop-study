#jdk

#  命令行中JDK版本切换
```sh title:~/.bash_profile
# >>> JDK Config <<<
# 1. 修正版本号写法
export JAVA_8_HOME=$(/usr/libexec/java_home -v 1.8 2>/dev/null)
export JAVA_11_HOME=$(/usr/libexec/java_home -v 11 2>/dev/null) # 去掉 1.

# 2. 默认设置为 Java 8
export JAVA_HOME=$JAVA_8_HOME

# 3. 这里的 PATH 这样写只能保证“初始”生效
export PATH=$JAVA_HOME/bin:$PATH

# 4. 【核心修改】别名里不仅要改 JAVA_HOME，还要更新 PATH
# 否则当你切换时，PATH 里旧的路径依然在前面
alias jdk8="export JAVA_HOME=\$JAVA_8_HOME && export PATH=\$JAVA_HOME/bin:\$PATH && echo 'Switched to JDK 8' && java -version"
alias jdk11="export JAVA_HOME=\$JAVA_11_HOME && export PATH=\$JAVA_HOME/bin:\$PATH && echo 'Switched to JDK 11' && java -version"
```