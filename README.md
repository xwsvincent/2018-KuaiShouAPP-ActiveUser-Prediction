# 2018-_HighestRankingTOP43
    2018中国高校计算机大赛——大数据挑战赛（以下简称“大赛”）是由教育部高等学校计算机类专业教学指导委员会、
        教育部高等学校软件工程专业教学导委员会教育部高等学校大学计算机课程教学指导委会、全国高等学校计算机教育研究会主办,
        由清华大学和北京快手科技有限公司联合承办，以脱敏和采样后的数据信息为基础开展的高端算法竞赛。 
        本次大赛基于脱敏和采样后的数据信息，预测未来一段时间活跃的用户。
        第一次独自打比赛，虽然成绩没有很突出，但是学习到了很多。
# 1.**模型思路**
    原始数据情况：只给出了用户1~30天的活动日志，要预测出31~37天内的活跃用户（在任意一个日志里出现过都算是活跃）。
    模型分析：本题是基于时间序列的未打标数据建模，使用滑窗法分割时间序列构造训练集。
    本次采用的滑窗法训练集时间划分如下：
          线下：
          训练集1：1~15（15） 测试集1：16~19（4）
          训练集2：1~19（19） 测试集2：20~24（5）
          训练集3：1~24（24） 测试集3：25~30（6）
          线上测试集：
          1~30
          线下训练集从对应测试集所属时间段的行为日志里获取标签，我采取训练集时间段/测试集时间段=4左右的划分方法，
          保证各个训练集与测试集的时间比例
# 2.**特征工程**
    原始用户日志只给出了10个属性（去掉相同的user_id），从以下几个方面去构造特征，我最后的特征数为105维
  ## （1）**基本的行为统计特征**：
        登录次数、拍摄次数、均值、单日最大值、方差、观看视频数等等，大概可以得到30~50个特征，虽然某些基本的行为统计特征
        对于算法划分模型并没有很明显的作用（使用RF、XGB进行特征选择可以看出某些行为特征基本没有被划分过），但是也不能轻易删除。
  ## （2）**基于时间序列的特征，关注用户最近的喜好**
  **本次比赛的核心：如何描述已注册的用户最近是更喜欢用快手还是最近更不喜欢用快手了。**
        显然，最近比以往更早时期更喜欢用快手的人我们会将其划分为活跃用户，相反，最近使用频率比更早时期要低，那我们更喜欢将其划分为不活跃用户。最理想的是，我们能让算法先感知到用户最近与更早时期的差距，再得知用户最近喜好趋势之后，再根据以往的行为统计特征进行判断。<br>
        比如：某用户以前每天都看几百个视频，但是最近只看几十个，我们希望算法先将其识别为喜好下降用户，再根据以往行为判定，及时该用户喜好下降，他未来七天也是活跃用户；或者，某用户注册二十几天，只有注册的那天有少量数据，但是最近七天却登录了三天，我们也希望算法能学习到怎么去判别此类用户。<br>
        上面说的就是**时间序列特征用来描绘用户在注册到30天这段时间使用APP趋势的作用，再加上基本的行为统计特征，可以保证分数0.813+。**<br>
        具体的时间序列特征包括：连续登录次数、最近间隔、活跃区间（30-注册）、最近一次连续登录天数与之前连续登录天数均值之间的比值\方差、最近一次间隔天数与总不活跃天数比值等等。
  ## （3）**关注能将人群进行划分的特征**
        所有使用快手APP的用户是一个大团体，团体之间总会通过某种关系形成不同的群体，相同群体里面的用户，
        在未来活跃不活跃这点上总是相同的。能将社交人群划分的特征有以下:
   ### ①device_type与register_type:
        相比于register_type只有11个取值，有数千个取值的device_type大家可能会觉得没有什么用，因为很多device_type只有一个人，
        其实我们可以将只有一个人的device_type划分到一起，再与register_type相结合，就会发现很重要的信息
        ————————某种device_type完全是僵尸粉，某种device_type的用户只出现了一次。          
  出现这种现象，应该是快手的广告产生了用户引流作用。
         比如：你在微博上不小心点了快手的广告，自动跳转到快手，那自然就会出现在登录日志里面（当然你得先有快手的账号）。
   ### ②每个视频的作者auhtor_id：
        原理很简单，大家都喜欢看的视频上传者，一般都是活跃用户；喜欢看相同作者发布的视频的用户，一般都有相同的喜好。
   ### ③ 添加人工特征：
        无论是测试集还是训练集，都可以发现，其实行为总数、拍摄总数都为0的用户都占了50%左右，
        所以添加判别行为以及拍摄总数是否为零的人工特征，对于提高算法成绩还是有一定效果的。
# 3.特征工程代码说明：
        文件中给出了获取连续登录次数、最后一次连续登录次数与之前连续登录次数的均值之间的方差与比值等代码。
# 4.算法选型： Xgboost
        试过tensorflow用BP、以及各种集成学习方法，Xgb效果最好。
        RNN、LSTM没有试过，感觉在本次比赛里深度模型非常容易过拟合，调参也很吃力。
        
    
  
