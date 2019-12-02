1. inv() 求矩阵的逆

2. format short/long 短/长显示

3. *矩阵乘法

   .*逐元素乘法

4. 复数虚部用i或者j表示

5. 冒号（:）用来引用多个元素

   特殊格式：start:step:end

6. whos可以用来查看工作区内容

7. num2str/int2str 将数字转换为字符

8. plot(x,y,'r--')来进行二维绘图，绘制出红色的虚线

   - ':' 绘制点线

   - 'r:+' 绘制红色+的线

   xlabel()绘制x轴标注

   ylabel()绘制y轴标注

   title()设置图标题

   使用hold on和hold off来绘制多条曲线，使用hold on以后再次绘制图像会在原图上叠加，直到使用hold off或者关闭

   legend()绘制图例

9. [meshgrid()函数](https://jingyan.baidu.com/article/d2b1d1029f82bb5c7f37d45d.html)

10. surf(x,y,z)进行三维绘图

    mesh(x,y,z)进行没有面的三维绘图

11. cylinder()返回r和h都为1的圆柱坐标值，沿圆柱体周长有20个等距分布的点

12. subplot(x,y,k) 生成一个总数为x*y，当前编辑图为k的子图集。

13. mean()求每列矩阵均值

14. diag()求对角线元素并生成列向量

15. sum() 求列向量和

    sum(,2)求行向量和
