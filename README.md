
package com.demo.action;

import java.io.IOException;
import java.io.PrintWriter;
import java.util.Vector;
import java.util.Date;
import java.text.SimpleDateFormat;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import com.demo.bean.ShopBean;
import com.demo.bean.MyOrderBean;
import com.demo.dao.BookDao;
import com.demo.bean.OrderItemBean;

public class AddOrderServlet extends HttpServlet {

	/**
	 * The doGet method of the servlet. <br>
	 *
	 * This method is called when a form has its tag value method equals to get.
	 * 
	 * @param request the request send by the client to the server
	 * @param response the response send by the server to the client
	 * @throws ServletException if an error occurred
	 * @throws IOException if an error occurred
	 */
	public void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {

		doPost(request,response);
	}

	/**
	 * The doPost method of the servlet. <br>
	 *
	 * This method is called when a form has its tag value method equals to post.
	 * 
	 * @param request the request send by the client to the server
	 * @param response the response send by the server to the client
	 * @throws ServletException if an error occurred
	 * @throws IOException if an error occurred
	 */
	public void doPost(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {

		response.setContentType("text/html");
		PrintWriter out = response.getWriter();
		
		//1.向myOrder表(订单表)添加订单记录
		//获取登录的用户名
		HttpSession session = request.getSession(true);  //创建一个session会话对象
		String userName = (String)session.getValue("user");
		
		//计算总价
		Vector booklist = (Vector)session.getValue("shopcar");  //将购物车中的商品取出送到向量数据类型中
		float priceNum = 0;
		for(int i = 0;i < booklist.size();i++){
			ShopBean book = (ShopBean)booklist.elementAt(i);//取出向量的第i个元素
			
			priceNum += book.getPrice() * book.getBuyNum();
		}
		
		//获取系统当前日期时间
		Date currDate = new java.util.Date();
		SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		String dateStr = sdf.format(currDate);
		
		//封装数据(打包)
		MyOrderBean orderBean = new MyOrderBean(userName,priceNum,dateStr);
		//添加记录(调用dao中的方法)
		boolean bool = new BookDao().addMyOrder(orderBean);
		
		//2.向orderItem表(订单明细表)添加记录
		//查询订单号(order_id)
		int orderId = new BookDao().searchOrder_id(userName, dateStr) ;
		
		for(int i = 0;i < booklist.size();i++){
			ShopBean book = (ShopBean)booklist.elementAt(i);//取出向量的第i个元素
			
			String isbn = book.getIsbn();
			int book_Num = book.getBuyNum();
			
			//封装数据
			OrderItemBean bean = new OrderItemBean(orderId,isbn,book_Num);
			//调用dao中的方法
			boolean bool2 = new BookDao().addMyOrderItem(bean);
		}
		
		if(bool){
			response.sendRedirect("BookListServlet");  //页面重定向
		}else{
			response.sendRedirect("index..jsp"); 
		}
		
	}

}
