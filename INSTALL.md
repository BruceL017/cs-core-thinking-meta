# 安装与使用指南

## 方式一：符号链接（推荐，便于同步更新）

```bash
# 1. 克隆仓库
git clone https://github.com/BruceL017/cs-core-thinking-meta.git

# 2. 进入仓库目录
cd cs-core-thinking-meta

# 3. 将所有 skills 符号链接到 Claude Code 的 skills 目录
for dir in */; do
  if [ -f "$dir/SKILL.md" ]; then
    base=$(basename "$dir")
    ln -sf "$(pwd)/$base" "$HOME/.claude/skills/$base"
  fi
done

# 4. 单独注册 router skill
ln -sf "$(pwd)/cs-core-thinking-router" "$HOME/.claude/skills/cs-core-thinking-router"
```

## 方式二：直接复制

```bash
git clone https://github.com/BruceL017/cs-core-thinking-meta.git
cd cs-core-thinking-meta

cp -r */ "$HOME/.claude/skills/"
```

## 验证安装

重启 Claude Code 后，输入任意相关话题（例如："系统流量翻倍该怎么办？"），agent 应该自动触发 `cs-core-thinking-router`。

## 更新

如果使用符号链接方式，只需在仓库目录执行 `git pull`，Claude Code 会自动加载最新版本的 skills。
