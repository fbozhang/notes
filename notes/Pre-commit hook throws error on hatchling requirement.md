




我正在按照[pre-commit 文档](https://pre-commit.com/)安装 git 预提交挂钩，以便将 **python** 代码格式从 **black** 格式化到我的 **Flask** 存储库。我已将 `pre-commit` 添加到我的 `requirements.txt` 中，我的 `pre-commit-config.yaml` 文件如下所示

```python
default_stages: [commit]
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v2.1.0
    hooks:
    - id: check-merge-conflict
  - repo: https://github.com/psf/black
    rev: stable
    hooks:
    - id: black
      language_version: python3.6
  - repo: https://github.com/pycqa/isort
    rev: 5.6.4
    hooks:
      - id: isort
        args: ["--profile", "black", "--filter-files"]
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v2.1.0
    hooks:
    - id: flake8
  - repo: https://github.com/alessandrojcm/commitlint-pre-commit-hook
    rev: v2.2.0
    hooks:
      - id: commitlint
        stages: [commit-msg]
        additional_dependencies: ['@commitlint/config-conventional']

```

使用 `pip install pre-commit` 安装 `pre-commit` 并使用 `pre-commit install` (相当于初始化，一次运行永久使用)，安装 **git hook** 脚本后，我收到与 `hatchling`	 依赖包相关的错误消息使用 `git commit`	 进行 **git** 提交。


```python
[INFO] Installing environment for https://github.com/psf/black.
[INFO] Once installed this environment will be reused.
[INFO] This may take a few minutes...
An unexpected error has occurred: CalledProcessError: command: ('/Users/vcui/.cache/pre-commit/repoq7qnm1w0/py_env-python3.6/bin/python', '-mpip', 'install', '.')
return code: 1
expected return code: 0
stdout:
    Processing /Users/vcui/.cache/pre-commit/repoq7qnm1w0
      Installing build dependencies: started
      Installing build dependencies: finished with status 'error'

stderr:
      ERROR: Command errored out with exit status 1:
       command: /Users/vcui/.cache/pre-commit/repoq7qnm1w0/py_env-python3.6/bin/python /Users/vcui/.cache/pre-commit/repoq7qnm1w0/py_env-python3.6/lib/python3.6/site-packages/pip install --ignore-installed --no-user --prefix /private/var/folders/9r/jcsx6jv923lb_fmm4w09zq5m0000gp/T/pip-build-env-wtkwct01/overlay --no-warn-script-location --no-binary :none: --only-binary :none: -i https://pypi.org/simple -- 'hatchling>=1.8.0' hatch-vcs hatch-fancy-pypi-readme
           cwd: None
      Complete output (2 lines):
      ERROR: Could not find a version that satisfies the requirement hatchling>=1.8.0 (from versions: 0.8.0, 0.8.1, 0.8.2, 0.9.0, 0.10.0, 0.11.0, 0.11.1, 0.11.2, 0.11.3, 0.12.0, 0.13.0, 0.14.0, 0.15.0, 0.16.0, 0.17.0, 0.18.0, 0.19.0, 0.20.0, 0.20.1, 0.21.0, 0.21.1, 0.22.0, 0.23.0, 0.24.0, 0.25.0, 0.25.1)
      ERROR: No matching distribution found for hatchling>=1.8.0
```

**原因**：`python3.6` - **pip** 无法找到符合您请求的，因为 **3.6** 支持在 **0.25.1** 之后被删除


你可以通过 	`language_version` 强制使用不同的语言版本（假设你安装了 **python**）——例如

```python
    - id: black
      language_version: python3.11
```
