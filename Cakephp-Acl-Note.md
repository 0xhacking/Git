Cakephp Acl（访问控制列表对用户的权限进行控制）

测试环境: CakePHP v2.5.1

### 1 在 /var/www/app 的根目录下导入 DbAcl:
------------------------------------------

###### $ cake schema create DbAcl

这里就是把 ``/var/www/app/Model/Config/Schema/db_acl.php`` 将 SQL 语句导入数据库。


    root@linux:/var/www/app# cake schema create DbAcl
    Failed loading /usr/lib/php5/20090626/xcache.so:  /usr/lib/php5/20090626/xcache.so: cannot open shared object file: No such file or directory

    Welcome to CakePHP v2.5.1 Console
    ---------------------------------------------------------------
    App : app
    Path: /var/www/app/
    ---------------------------------------------------------------
    Cake Schema Shell
    ---------------------------------------------------------------

    The following table(s) will be dropped.
    acos
    aros
    aros_acos
    Are you sure you want to drop the table(s)? (y/n)
    [n] > y
    Dropping table(s).
    acos updated.
    aros updated.
    aros_acos updated.

    The following table(s) will be created.
    acos
    aros
    aros_acos
    Are you sure you want to create the table(s)? (y/n)
    [y] > y
    Creating table(s).
    acos updated.
    aros updated.
    aros_acos updated.
    End create.``



### 2 User Model 下添加如下代码：
-------------------------------

    <?php
    class User extends AppModel
    {
        public $name = 'User';
        public $actsAs = array('Acl' => array('type' => 'requester'));
        function parentNode() { return "Users"; }
        ...
    }
    ?>

###### $ cake acl view aro 

    root@linux:/var/www/app# cake acl view aro
    Failed loading /usr/lib/php5/20090626/xcache.so: /usr/lib/php5/20090626/xcache.so: cannot open shared object file: No such file or directory

    Welcome to CakePHP v2.5.1 Console
    ---------------------------------------------------------------
    App : app
    Path: /var/www/app/
    ---------------------------------------------------------------
    Aro tree:
    ---------------------------------------------------------------
    [2] Users
    [3] Users
    [4] Users
    [5] Users
    [6] Users
    [7] Users
    [8] Users
    [9] Users
    [10] Users
    [11] User.30
    [12] User.31
    ---------------------------------------------------------------

###3 Post Model 添加 ACOS:
--------------------------

    <?php
    class Post extends AppModel
    {
        public $name = 'Product';
        public $actsAs = array('Acl' => array('type' => 'controlled'));
        function parentNode() { return 'Post';}
        ...
    }
    ?>

###4 PostsController 添加 ACOS:
------------------------------

    <?php
    class PostsController extends AppController {

        public $name = 'Posts';
        public $components = array('Acl');
        ...
    ?>

###5 add():
--------

    <?php
    function add() {
        if ($this->Product->save($this->data)) {

            $this->Acl->allow( 'Users', array('model'=>'Product', 'foreign_key' => $this->Product->id), 'read');
            $this->Acl->allow( array('model'=>'User', 'foreign_key' => $this->Session->read('user_id') ), array('model'=>'Product', 'foreign_key' => $this->Product->id));

            $this->Session->setFlash(__('The product has been saved', true));
            $this->redirect(array('action' => 'index'));
        } else {

            $this->Session->setFlash(__('The product could 
                not be saved. Please, try again.', true));
        }

        $dealers = $this->Product->Dealer->find('list');
        $this->set(compact('dealers'));
    }
    ?>

###6 view():
---------

    <?php
    function view($id = null) {
        if ($this->Acl->check( array('model' => 'User', 'foreign_key' => (int) $this->Session-> read('user_id')),array('model' => 'Product', 'foreign_key' => $product['Product']['id']), 'read')) {
            $this->set('product', $product);

        } else {
            $this->Session->setFlash('Only registered users may view this product.');
            $this->redirect(array('action'=>'index'));
        }
    }
    ?>

###7 edit():
---------

    <?php
    function edit($id = null) {  
            if (@$this->Acl->check( array('model' => 'User', 'foreign_key' => (int) $this->Session->read('user_id')), array('model' => 'Product', 'foreign_key' => $product['Product']['id']), 'update')) {

                if ($this->Product->save($this->data)) {
                    $this->Session->setFlash(__('The product has been saved', true));
                    $this->redirect(array('action' => 'index'));
                } else {
                    $this->Session->setFlash(__('The product could not be saved. Please, try
                        again.', true)$
                }

            } else {

                $this->Session->setFlash('You may not edit this product.');
                $this->redirect(array('action'=>'index'));
            }
            $this->set('product', $product);
        }
    ?>

###8 delete():
-----------

    <?php
     function delete($id = null) {
            if (@$this->Acl->check( array('model' => 'User', 'foreign_key' => (int) $this->Session-> read('user_id')),
                array('model' => 'Product', 'foreign_key' => $product['Product']['id']), 'delete')) {

                    if ($this->Product->delete($id)) {
                        $this->Session->setFlash(__('Product deleted', true));
                        $this->redirect(array('action'=>'index'));
                    }

                    $this->Session->setFlash(__('Product was not deleted', true));
                    $this->redirect(array('action' => 'index'));

                } else {
                    $this->Session->setFlash('You may not delete this product.');
                    $this->redirect(array('action'=>'index'));
                }
    }
    ?>
