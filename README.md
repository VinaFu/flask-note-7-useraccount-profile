# flask-note-7-useraccount-profile
用户如何修改post文和头像


    <div class="content-section">
        <div class="media">
        <img class="rounded-circle account-img" src="{{ image_file }}"> 改图像路径
        <div class="media-body">
            <h2 class="account-heading">{{ current_user.username}}</h2> 改成用户登陆名
            <p class="text-secondary">{{ current_user.email}}</p> 改成用户邮箱
        </div>
        </div>

1. 改头像

  1.1 routes.py
  
  def account ():
    image_file = url_for('static', filename='profile_pics/' + current_user.image_file)
    return render_template('account.html', title='Account', image_file=image_file)
    加入了默认图像，之前models里面定义了profile就是用了默认图片且放在了profile里面。所以可以直接引用。





