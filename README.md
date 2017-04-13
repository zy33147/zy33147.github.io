在yii2 中利用yii\filters\VerbFilter简单的防止前台不停的点击请求按钮

首先在你的models下新建一个OrderFilter文件,继承yii\base\ActionFilter

class OrderFilter extends ActionFilter
{
    public function beforeAction($action)
    {
        //写你要控制的方法(控制器/方法名)
        $arr = [
            'order/refunding',
        ];
        if ( in_array($action->controller->id.'/'.$action->id, $arr) ){
            if (!empty(Yii::$app->request->post())) {
                $post_data = Yii::$app->request->post();
                if ($this->session->has(md5($post_data['_csrf']))) {
                    Yii::$app->session->setFlash('success', Yii::t('退款', '提交速度太快，请刷新页面'));
                    Yii::$app->getResponse()->redirect(Yii::$app->request->getReferrer());
                    return false;
                }
                Yii::$app->session->set(md5($post_data['_csrf']),uniqid());
                session_write_close();//session写入的时候会等待整个页面加载完成，这里用此函数可以立即写入
            }
        }
        
        return parent::beforeAction($action);
    }
}

然后再你控制的behaviors配置上

   return [
            'verbs' => [
                'class' => VerbFilter::className(),
                //要使用的方法
                'actions' => [
                    'refunding' => ['POST'],
                ],
            ],
            //之前创建的filter文件
            'order' => [
                'class' => 'backend\models\OrderFilter',
            ],
      ];

