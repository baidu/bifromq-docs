---
sidebar_position: 22
---

# 配置规范与迁移
配置项在不同版本之间可能会有所不同。在部署BifroMQ时，您可以使用之前的配置进行设置，这不会中断BifroMQ的运行。然而，请注意这些配置可能不会生效。

此外，在`info.log`的开始处，会打印完整的配置。

为了简化BifroMQ的设置和定制，它使用standalone.yml中提供的值作为部分常用可配置项的设置值。对于在standalone.yml中未指定的设置，将采用默认值以确保系统在无需手动干预的情况下正确运行。

重要的是要认识到，虽然BifroMQ不保证不同版本之间配置项的兼容性，但它旨在确保旧版本的配置不会导致新版本BifroMQ的功能出现中断或异常。这种方法旨在最小化由于版本差异可能引起的潜在问题，从而促进更平滑的运行体验。

为了进一步简化版本升级过程中的迁移工作，BifroMQ在启动时会将standalone.yml文件的完整内容输出到`info.log`文件中。这一特性使用户可以方便地通过检查日志文件中的差异来比较和更新他们的standalone.yml配置。通过提供正在使用的配置设置的详细记录，BifroMQ帮助用户维护一个最新且高效的部署，确保系统针对他们的特定需求进行了优化。