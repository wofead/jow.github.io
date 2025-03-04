



# 资源工作流程-资源导入导出流程

[toc]



Cocos Creator 是专注于内容创作的游戏开发工具，在游戏开发过程中，对于每个项目该项目专用的程序架构和功能以外，我们还会生产大量的场景、角色、动画和 UI 控件等相对独立的元素。对于一个开发团队来说，很多情况下这些内容元素都是可以在一定程度上重复利用的。

在以场景和 Prefab 为内容组织核心的模式下，Cocos Creator 内置了场景（`.fire`) 和预制 (`.prefab`) 资源的导出和导入工具。

## 资源导出

在主菜单选择 **文件 -> 资源导出**，即可打开资源导出工具面板，接下来可以用以下两种方式选择需要导出的资源：

- 将场景或预制文件从 **资源管理器** 中拖拽到导出资源面板的资源栏中
- 点击资源栏右边的 **选择** 按钮，打开文件选择对话框，并在项目中选取你要导出的资源

可以选择的资源包括 `.fire` 场景文件和 `.prefab` 预制文件。

## 确认依赖

导出工具会自动检查所选资源的依赖列表并列出在面板里，用户可以手动检查每一项依赖是否必要，并剔除部分依赖的资源。被剔除的资源将不会被导出。

确认完毕后点击 **导出** 按钮，会弹出文件存储对话框，用户需要指定一个文件夹位置和文件名，点击 **存储**，就会生成 `文件名.zip` 的压缩包文件，包含导出的全部资源。

## 资源导入

有了导出的资源包，就可以在新项目中导入这些现成的资源了，在新项目的主菜单里选择 **文件 -> 导入资源**，即可打开资源导入面板。

点击 **Zip 文件路径** 输入框右边的 **选择** 按钮，在文件浏览对话框中选择刚才导出的导出资源压缩包。

导入过程中也会让用户再次确认导入资源依赖，这时候可以通过取消某些资源的勾选来去掉不需要导入的资源。

## 设置导入位置

相比导出过程，导入过程中增加了 `导入目标路径` 的设置，用户可以点击旁边的 **选择** 按钮，选择项目的 `assets` 路径下的某个文件夹作为导入资源的放置位置。由于导出资源时所有资源的路径都是以相对于 `assets` 路径来保存的，导入时如果不希望导入的资源放入 `assets` 根目录下，就可以再指定一层中间目录来隔离不同来源的导入资源。

设置完成后点击 **导入** 按钮，会弹出确认对话框，确认后就会把列出的资源导入到目标路径下。

### 脚本和资源冲突

由于 Creator 项目中的脚本不能同名，当导入的资源包含和当前项目里脚本同名的脚本时，将不会导入同名的脚本。如果出现导入资源的 UUID 和项目中现有资源 UUID 冲突的情况，会自动为导入资源生成新的 UUID，并更新在其他资源里的引用。

