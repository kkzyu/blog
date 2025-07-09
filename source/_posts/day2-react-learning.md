---
layout: posts
title: day2-react-learning
date: 2025-07-09 10:04:27
tags: [react, web前端]
---
用模拟数据写了一个作业管理的详情页，react可以用mock拦截前端发出去的请求，所以可以在这里放置模拟数据。
给了一个类似这样的界面：
![alt text](day2-react-learning-1.png)
1. 首先确定了把整个页面拆分成三个组件（上basicinfocard下documenttablecard），还有一个根组件jobdetail来负责获取jobid，管理整体的加载状态并组织子组件的渲染。
2. 接着是和后端的交互：
- 用useEffect Hook，在组件挂载时根据传入的jobid调用service文件夹下job.ts中定义的接口函数getJobDocuments来从后端异步获取文档列表数据。
- 使用useState创建dataSource状态，用于存储从后端获取的完整文档列表。获取到数据之后，对每条记录增加一个唯一的key属性来满足protable的要求。
*protable功能确实挺丰富，部署起来也挺快捷的。这里先把它稍微进行了一下更改再拿来用，学习文档：https://procomponents.ant.design/components/table*
3. protable的核心交互功能：
- 行选择和批量下载：
    1. 创建了一个selectedRowKeys状态数组用来存放所有被勾选的行的key。
    2. 该列用render的方法返回一个<Checkbox>，其勾选状态与selectedRowKeys双向绑定。
    3. 该列的 title 属性也返回一个 <Checkbox>，作为“全选/取消全选”的控制器。其 checked 和 indeterminate (半选) 状态根据 selectedRowKeys 和 dataSource 的长度动态计算，实现了精确的全选逻辑。
- 在名称列，将其渲染为一个 <Button type="link">，使其外观像一个可点击的链接。用<Modal> 窗口来显示日志内容（用setLogModalVisible来控制）。
- 把protable的搜索功能取消（因为组件样式的自定义改动空间不大），自定义了一个<Input.Search> 组件。其 onSearch 事件会更新 searchText 状态。创建一个派生变量 filteredDataSource。它通过对原始的 dataSource 数组使用 .filter() 方法，根据 searchText 的值进行客户端实时过滤。ProTable 的 dataSource 属性直接使用这个过滤后的数据，从而实现了搜索功能。

学习到的新知识：
1. Hooks：
    1. useState：相当于内部记忆（可修改的便签纸），用于为函数组件添加状态。
    比如const [logModalVisible（便签纸）, setLogModalVisible（专用于修改便签纸的笔）] = useState(false)，其中logModalVisible的状态就是false。我们可以并只能通过调用setLogModalVisible(true)来更新便签纸的状态。
    & useParams：相当于外部指令（门牌号），来自于网页的url地址，而不是内部的组件。它只能读取而无法控制。
    2. useEffect：用于在特定条件下，自动执行一些份外的任务（比如和外界沟通）。类似于一条如果就的指令。比如说const { jobId } = useParams();就相当于看了一眼网页地址的门牌号，但是无法修改门牌。
    比如useEffect(() => { fetchData(); }, [jobId]);
    其中fetchData()意思是去服务器中拿回文件列表，jobid就是触发的ID（如果这个id发生了改变就去服务器拿回文件列表，但是如果还是同一个id的话就不需要跑一趟，用现有的就行。）
    注意：①如果指令的触发条件是[]，意思即使开机后只执行一次以后都不管了；②如果没有触发条件，意思是任何一点风吹草动都要去执行这个指令。
    3. useContext：相当于公共广播系统。顶层组件可以通过广播发布信息，比如广播“当前主题：夜间模式”，那么所有组件（深层的也可以）只要打开收音机都会听到这个信息。
    4. useMemo：用于记住一个复杂的计算结果（重在结果），避免重复计算。
    5. useCallback：用于记住一个计算公式（重在技能）。这在传递给子组件时非常有用。
    6. useRef：相当于多功能工具箱，无论怎么刷新箱子里的东西都是同一个不会丢失。修改箱子里的东西也不会触发重新渲染。