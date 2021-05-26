# Docker 设置 CI/CD

## CI/CD 最佳实践
### 使用 Docker Hub 进行 CI/CD 作为最佳实践
根据[2020 Jetbrains developer survey](https://www.jetbrains.com/lp/devecosystem-2020/)调查，44% 的开发者正在使用 Docker 容器进行各种形式的持续集成和持续发布。我们理解大量的开发者将 Docker Hub 设置为容器仓库，作为他们 CI/CD 工作流的一部分。这篇教程如何使用 Docker 作为 CI/CD 的最佳实践。

我们听到一些反馈关于网络出口和免费用户拉去镜像的次数限制的改变，这对于没有限制的将 Docker Hub 作为 CI/CD 工作流的一部分，还有一些问题。这个教程涵盖的最佳实践可以提升你经验以及如何合理使用 Docker Hub 来奖励限制的风险，同时也包含了一些如何增加限制的提示。

### 内部和外部循环
首先，使用 Docker 和任何 CI/CD 时最重要的事情之一就是了解何时需要使用 CI 进行测试，以及何时可以在本地进行测试。在 Docker 上，我们考虑开发者如何在内循环工作(编码，构建，运行和测试)