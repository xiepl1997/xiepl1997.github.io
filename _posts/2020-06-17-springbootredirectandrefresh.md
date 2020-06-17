---
layout : post
title : springboot中的重定向数据传递
date : 2020-06-17
author : xiepl1997
tags : springboot
---

在写springboot项目的时候，有时候会有重定向的需求，例如登录界面登录后，就应该使用**重定向**来进行页面的跳转。如果这时候使用的是转发的方式来进行页面的跳转的话，会出现两个问题：
* 浏览器上的路径不会改变
* 在主页中点击刷新时，页面会提示需要再次提交表单
因为转发是通过forward()方法提交信息在多个页面之间进行传递。  

登陆后地址栏是需要变为主页地址的，而且也不应该出现刷新提示提交表单的现象，所以应该使用重定向来进行登录跳转。那么这就出现了一个问题，重定向的页面不能读取转向前通过request.setAttribute()或者是ModelAndView设置的属性值，那怎么办呢?**这里可以使用RedirectAttribute类来保存之前获得的属性，这样就可以避免重定向之后数据丢失的问题了。**  
就是说，原来我们是向ModelAndView中放属性的，但是现在我们只要往这个类里面放就ok了，就可以让用户在主界面中看到之前想保存的一堆数据了。  

下面贴出一个例子，同时使用到了转发和重定向。还创建了cookie和session。
```java
    @RequestMapping("/login")
    public String login(@RequestParam("id")String id,
                        @RequestParam("name")String name,
                        RedirectAttributes redirectAttributes,
                        Model model,
                        HttpServletRequest request,
                        HttpServletResponse response) throws UnsupportedEncodingException {
        Student student = studentService.getStudentByID(id);
        String msg = "";
        if(student == null){
            msg = "学号不存在！";
            model.addAttribute("msg", msg);
            return "login";
        }
        else{
            if(!name.equals(student.getName())){
                msg = "学号或姓名输入错误！请检查！";
                model.addAttribute("msg", msg);
                return "login";
            }
        }
        //创建session
        HttpSession session = request.getSession();
        session.setAttribute("student", student);

        //创建cookie
        Cookie cookie = new Cookie("id", id);
        //cookie中存储中文需要转为utf-8
        Cookie cookie1 = new Cookie("name", URLEncoder.encode(name, "UTF-8"));
        cookie.setMaxAge(7*24*60*60); //保存时间一周
        cookie1.setMaxAge(7*24*60*60);

        response.addCookie(cookie);
        response.addCookie(cookie1);

        //如果账号是管理员账号
        if(id.equals("1111")){
            return "redirect:/adminpage.html";
        }

        //获取用户当天的填报数据
        Commit commit1 = commitService.getFirstCommitByIdToday(id);
        Commit commit2 = commitService.getSecondCommitByIdToday(id);
        Commit commit3 = commitService.getThirdCommitByIdToday(id);

        redirectAttributes.addFlashAttribute("commit1", commit1);
        redirectAttributes.addFlashAttribute("commit2", commit2);
        redirectAttributes.addFlashAttribute("commit3", commit3);

        redirectAttributes.addFlashAttribute("student", student);

        return "redirect:/index.html";
    }
```
从上面可以看到当登录成功的情况下，使用的是重定向来跳转主页面。同时使用了redirectAttributes.addFlashAttribute()方法来传递数据。  

同时redirectAttributes还有addAttribute()方法，下面总结这两个方法：  
1. addAttribute()方法直接把你的参数加到url的后面进行重定向，使用这个方法之后，虽然你成功的跳到了主界面，但是！页面的URL变成了：/test/Main?userName=10000&password=22222，一般的网站登录好像是没人这么干的吧，所以这就不做讲解了。而且要是想获取数据的话，可以直接用@RequestParam或者@ModelAttribute来获取。
2. 但是如果使用addFlashAttribute()方法就没有那么多事了，这个方法的使用和上面的那个是一样的，唯一的区别就是传输数据的方法不一样。**这个方法会把你加入的数据放在一个叫做FlashMap的Map中。在进行重定向的时候，这个Map中的数据会跟着新生成的Request一起到去请求重定向到的页面（也就是程序的的主界面），当重定向之后，这个Map里的数据会被清空，意思就是现在只有新页面的Request才有这些数据了。**

**当然，如果现在刷新一下页面，你会发现展示的数据（一开始你存储在FlashMap中的数据）没了。这其实也很好理解吧，刚刚说了，FlashMap在把数据给重定向生成的Request后，就把自己的数据删了。现在你刷新了一下页面相当于客户端又对这个URL产生了新请求，这个新请求是不可能有上一个请求中的信息的，所以避免了表单的重复提交。**  

在例子中，我的解决办法是当点击刷新index页面时，再次通过查询数据库获取数据后转发。，如下
```java
    /**
     * 刷新页面时
     * @param request
     * @param mv
     * @return
     */
    @RequestMapping("/index.html")
    public ModelAndView Refresh(HttpServletRequest request,
                                ModelAndView mv){
        HttpSession session = request.getSession();
        Object student = session.getAttribute("student");
        if(student == null){
            mv.setViewName("login");
            return mv;
        }

        mv.addObject("student", ((Student)student));

        String id = ((Student)student).getId();

        //获取当天的填报数据
        Commit commit1 = commitService.getFirstCommitByIdToday(id);
        Commit commit2 = commitService.getSecondCommitByIdToday(id);
        Commit commit3 = commitService.getThirdCommitByIdToday(id);

        mv.addObject("commit1", commit1);
        mv.addObject("commit2", commit2);
        mv.addObject("commit3", commit3);
        mv.addObject("student", (Student)student);
        mv.setViewName("index");

        return mv;

    }
```
其实这种方法也不是很好，因为数据实际上没有丢，，  

虽然说，管理FlashMap的管理器默认是SessionFlashMapManager，保存在Session中。但是，通过Session是无法获得数据的。  

**方法一：使用RequestContextUtils.getInputFlashMap()来直接获取FlashMap（重点）**

我们在Index-Controller中，把数据使用RedirectAttribute.addFlashAttribute()保存在FlashMap中，在重定向的时候，FlashMap中的数据被添加到了Request中，所以使用这个方法可以从Request中获取到FlashMap。

FlashMap本质上还是一个Map，所以这个方法的返回值是Map<String，?>，我们只要用一个Map来接收就可以了，然后我们就可以提取我们之前存储下的那些属性了。（就是可以直接在视图上使用 ${属性名} 来获取）。

**方法二：通过（@ModelAttribute（value="属性名"）String xxx）通过属性名称获取属性值（推荐）**

刚刚已经说过了，一开始储存的数据在重定向的时候都放在Request里了，这些数据被作为初始化的模型（通俗的来讲就是这些数据被这个控制器的所有方法共享，即为共享数据），所以通过@ModelAttribute可以在Main-Controller中的任意一个方法里通过属性名获取这些属性。比上面的那个要简单多了，所以推荐使用这个方法。