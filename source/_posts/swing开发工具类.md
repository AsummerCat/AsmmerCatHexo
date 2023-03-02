---
title: swing开发工具类
date: 2023-02-11 02:03:31
tags: [java,swing]
---

# swing开发工具类
<!--more-->
```
package com.linjingc.guidemo.util;

import cn.hutool.core.collection.CollectionUtil;
import org.springframework.util.StringUtils;

import javax.swing.*;
import java.awt.*;
import java.util.List;
import java.util.Optional;

public class SwingUtils {

    /**
     * 创建子窗体
     *
     * @return
     */
    public static JPanel createPlane() {
        JPanel p = new JPanel();
        //设置面板为流式布局居中显示，组件横、纵间距为5个像素
        p.setLayout(new FlowLayout(1, 5, 5));
        p.setBackground(Color.pink);
        return p;
    }

    /**
     * 创建子窗体
     *
     * @param bg 背景色
     * @return
     */
    public static JPanel createPlane(Color bg) {
        JPanel p = new JPanel();
        //设置面板为流式布局居中显示，组件横、纵间距为5个像素
        p.setLayout(new FlowLayout(1, 5, 5));
        p.setBackground(bg);
        return p;
    }

    /**
     * 创建瀑布子窗体
     *
     * @return
     */
    public static JPanel createGripPlane() {
        JPanel p = new JPanel(new GridLayout(3, 2, 10, 10));
        //3行2列 水平间距20 垂直间距10
        p.setBackground(Color.pink);
        return p;
    }

    /**
     * 创建瀑布子窗体
     *
     * @param bg 背景色
     * @return
     */
    public static JPanel createGripPlane(Color bg) {
        JPanel p = new JPanel(new GridLayout(3, 2, 20, 10));
        //3行2列 水平间距20 垂直间距10
        p.setBackground(bg);
        return p;
    }
    public static JPanel createGripPlane(GridLayout gridLayout) {
        JPanel p = new JPanel(gridLayout);
        p.setBackground(Color.pink);
        return p;
    }


    /**
     * 建子窗体移动到布局顶部
     *
     * @param jFrame 整体布局
     * @param p      子窗体
     *               North:顶部 South:底部  West:左侧 East:右侧  Center:居中
     */
    public static void moveUpWindows(JFrame jFrame, JPanel p) {
        jFrame.getContentPane().add("North", p);
    }

    /**
     * 建子窗体移动到布局底部
     *
     * @param jFrame 整体布局
     * @param p      子窗体
     *               North:顶部 South:底部  West:左侧 East:右侧  Center:居中
     */
    public static void moveDownWindows(JFrame jFrame, JPanel p) {
        jFrame.getContentPane().add("South", p);
    }

    /**
     * 建子窗体移动到布局左侧
     *
     * @param jFrame 整体布局
     * @param p      子窗体
     *               North:顶部 South:底部  West:左侧 East:右侧  Center:居中
     */
    public static void moveLeftWindows(JFrame jFrame, JPanel p) {
        jFrame.getContentPane().add("West", p);
    }

    /**
     * 建子窗体移动到布局右侧
     *
     * @param jFrame 整体布局
     * @param p      子窗体
     *               North:顶部 South:底部  West:左侧 East:右侧  Center:居中
     */
    public static void moveRightWindows(JFrame jFrame, JPanel p) {
        jFrame.getContentPane().add("East", p);
    }

    /**
     * 建子窗体移动到布局右侧
     *
     * @param jFrame 整体布局
     * @param p      子窗体
     *               North:顶部 South:底部  West:左侧 East:右侧  Center:居中
     */
    public static void moveCenterWindows(JFrame jFrame, JPanel p) {
        jFrame.getContentPane().add("Center", p);
    }


    /**
     * 子窗体添加按钮
     *
     * @param jPanel 子窗体
     * @param name   按钮名称
     * @return
     */
    public static JButton createJpanelButton(JPanel jPanel, String name) {
        JButton b = new JButton(name);
        jPanel.add(b);
        return b;
    }
    /**
     * 子窗体添加按钮带字体
     *
     * @param jPanel 子窗体
     * @param name   按钮名称
     * @param font 字体
     * @return
     */
    public static JButton createJpanelButton(JPanel jPanel, String name,Font font) {
        JButton b = new JButton(name);
        b.setFont(font);
        jPanel.add(b);
        return b;
    }



    /**
     * 新增单行输入框
     *
     * @param jPanel    子窗体
     * @param writeFlag 是否允许编辑 true允许 false不可编辑
     * @param size      输入框长度字符
     * @param text      预输入文字
     * @return
     */
    public static JTextField createJpanelSingleText(JPanel jPanel, boolean writeFlag, int size, String text) {
        // 创建一个单行输入框
        JTextField textField = new JTextField();
        // 设置输入框允许编辑
        textField.setEditable(writeFlag);
        // 设置输入框的长度为size个字符
        textField.setColumns(size);
        if (!StringUtils.isEmpty(text)) {
            textField.setText(text);
        }
        jPanel.add(textField);
        return textField;
    }


    /**
     * 新增多行输入框
     *
     * @param jPanel    子窗体
     * @param writeFlag 是否允许编辑
     * @param size      单行输入框长度
     * @param height    多行
     * @param lineWrap  是否自动换行
     * @param text      预输入文字
     * @return
     */
    public static JTextArea createJpanelMulitText(JPanel jPanel, boolean writeFlag, int size, int height, boolean lineWrap, String text) {
        // 创建一个多行输入框
        JTextArea area = new JTextArea();
        // 设置输入框允许编辑
        area.setEditable(writeFlag);
        // 设置输入框的长度为14个字符
        area.setColumns(size);
        if (!StringUtils.isEmpty(text)) {
            area.setText(text);
        }
        // 设置输入框的高度为3行字符
        area.setRows(height);
        // 设置每行是否允许折叠。为true的话，输入字符超过每行宽度就会自动换行
        area.setLineWrap(lineWrap);
        jPanel.add(area);
        return area;
    }

    /**
     * 新增多行输入框+滚动条
     *
     * @param jPanel    子窗体
     * @param writeFlag 是否允许编辑
     * @param size      单行输入框长度
     * @param height    多行
     * @param lineWrap  是否自动换行
     * @param text      预输入文字
     * @return
     */
    public static JScrollPane createJpanelMulitTextScroll(JPanel jPanel, boolean writeFlag, int size, int height, boolean lineWrap, String text) {
        // 创建一个多行输入框
        JTextArea area = new JTextArea();
        // 设置输入框允许编辑
        area.setEditable(writeFlag);
        // 设置输入框的长度为14个字符
        area.setColumns(size);
        // 设置输入框的高度为3行字符
        area.setRows(height);
        if (!StringUtils.isEmpty(text)) {
            area.setText(text);
        }
        // 设置每行是否允许折叠。为true的话，输入字符超过每行宽度就会自动换行
        area.setLineWrap(lineWrap);
        // 创建一个滚动条
        JScrollPane scroll = new JScrollPane(area);
        jPanel.add(scroll);
        return scroll;
    }

    /**
     * 密码输入框
     *
     * @param jPanel    子窗体
     * @param writeFlag 是否允许编辑 true允许 false不可编辑
     * @param size      输入框长度字符
     * @return
     */
    public static JPasswordField createJpanelPasswordText(JPanel jPanel, boolean writeFlag, int size) {
        //密码输入框
        JPasswordField passwordField = new JPasswordField();
        // 设置密码框允许编辑
        passwordField.setEditable(writeFlag);
        // 设置密码框的长度为11个字符
        passwordField.setColumns(size);
        // 设置密码框的回显字符。默认的回显字符为圆点
        passwordField.setEchoChar('*');
        jPanel.add(passwordField);
        return passwordField;
    }

    /**
     * 创建label文字
     *
     * @param jPanel    子窗体
     * @param text      文字
     * @param alignment 位置
     * @return
     */
    public static JLabel createLabel(JPanel jPanel, String text, Integer alignment) {
        JLabel jl3 = new JLabel(text);
        if (!Optional.ofNullable(alignment).isPresent()) {
            jl3.setHorizontalAlignment(SwingConstants.RIGHT);
        } else {
            jl3.setHorizontalAlignment(alignment);
        }
        jPanel.add(jl3);
        return jl3;
    }
    /**
     * 创建label文字
     *
     * @param jPanel    子窗体
     * @param text      文字
     * @param alignment 位置
     * @param f 字体大小 Font f=new Font(Font.DIALOG,Font.BOLD,18);
     * @return
     */
    public static JLabel createLabel(JPanel jPanel, String text, Integer alignment,Font f) {
        JLabel jl3 = new JLabel(text);
        if (!Optional.ofNullable(alignment).isPresent()) {
            jl3.setHorizontalAlignment(SwingConstants.RIGHT);
        } else {
            jl3.setHorizontalAlignment(alignment);
        }
        jl3.setFont(f);
        jPanel.add(jl3);
        return jl3;
    }

    /**
     * 弹窗
     *
     * @param parentComponent frame 选择this
     * @param title           标题
     * @param message         提示语
     * @param icon            JOptionPane.QUESTION_MESSAGE 图标
     * @param data            数据集合
     * @param defaultValue    默认值
     * @return
     */
    public static Object createPopup(Component parentComponent, String title, String message, int icon, Object[] data, Object defaultValue) {
//        Object popup = SwingUtils.createPopup(this, "按键测试(电源,多媒体)", "请选择一项: ", JOptionPane.QUESTION_MESSAGE, new Object[]{"通过", "不通过"}, "通过");
        Object inputContent = JOptionPane.showInputDialog(
                parentComponent,
                message,
                title,
                icon,
                null,
                data,
                defaultValue
        );
        return inputContent;
    }

    /**
     * 新增导航栏
     *
     * @param jFrame
     * @return
     */
    public static JMenuBar createMenuBar(JFrame jFrame) {
        JMenuBar mb = new JMenuBar();
        jFrame.setJMenuBar(mb);
        return mb;
    }

    /**
     * 新增菜单
     *
     * @param bar      导航栏
     * @param name     菜单名称
     * @param itemList 菜单项 new JMenuItem("近战-Warriar")
     */
    public static void addMenu(JMenuBar bar, String name, List<JMenuItem> itemList) {
        JMenu mHero = new JMenu(name);
        if (CollectionUtil.isNotEmpty(itemList)) {
            for (JMenuItem jMenuItem : itemList) {
                mHero.add(jMenuItem);
                //分隔符
                mHero.addSeparator();
            }
        }
        bar.add(mHero);
    }


    /**
     * 创建外部弹窗
     *
     * @param jFrame
     */
    public static JDialog createDialog(JFrame jFrame) {
        // 根据外部窗体实例化JDialog 需要关闭后才会显示原来的窗体
        JDialog d = new JDialog(jFrame);
        // 设置为模态
        d.setModal(true);
        d.setTitle("模态的对话框");
        d.setSize(400, 300);
        d.setLocation(200, 200);
        d.setLayout(null);
        JButton b = new JButton("一键秒对方基地挂");
        b.setBounds(50, 50, 280, 30);
        d.add(b);
        d.setVisible(true);
        return d;
    }

    /**
     * 新增工具栏
     *
     * @param buttons 按钮
     * @return
     */
    public static JToolBar createToolBar(List<JButton> buttons) {
        //新增工具栏
        JToolBar tb = new JToolBar();
        if (CollectionUtil.isNotEmpty(buttons)) {
            for (JButton button : buttons) {
                tb.add(button);
            }
        }
        // 禁止工具栏拖动
        tb.setFloatable(true);
        // 把工具栏放在north的位置
        tb.setLayout(new BorderLayout());
        return tb;
    }


    /**
     * 创建表格
     * @param columnNames 表头
     * @param data 表格数据
     * @param editFlag
     * @return
     */
    public static JTable createTable(String[] columnNames, String[][] data, boolean editFlag) {
//        JTable t = SwingUtils.createTable(columnNames, heros, false);
//        // 当选择了某一行的时候触发该事件 使用selection监听器来监听table的哪个条目被选中
//        t.getSelectionModel().addListSelectionListener(
//                e -> {
//                    int rows = t.getRowCount(); //获取多少行
//                    int cols = t.getColumnCount(); //获取多少列
//                    int index1 = t.getSelectedRow();//获取选中行
//                    int column1 = t.getSelectedColumnCount();//获取选中列
//                    //获取选中行数据
//                    for (int i = 1; i < cols; i++) {
//                        System.out.print(t.getValueAt(index1, i));
//                        System.out.print("----");
//                    }
//                    System.out.println("输出完成");
//
//                });

//        String[] columnNames = new String[]{"id", "name", "hp", "damage"};
//        String[][] heros = new String[][]{
//                {"1", "盖伦", "616", "100"},
//                {"2", "提莫", "512", "102"},
//                {"3", "奎因", "832", "200"}
//        };
        JTable t = new JTable(data, columnNames) {
            //设置表格禁止修改
            @Override
            public boolean isCellEditable(int row, int column) {
                if (editFlag) {
                    return super.isCellEditable(row, column);
                }
                return false;
            }
        };
        // 设置列宽度
        t.getColumnModel().getColumn(0).setPreferredWidth(10);
        //JScrollPane 用来显示表头
//        JScrollPane sp = new JScrollPane(t);
        return t;
    }



}

```