# OGRE-
初级教程4
//Ogre基础教程4，帧监听（用鼠标控制观察方向有点问题），2015.10.4 

#include "ExampleApplication.h"

class TutorialFrameListener:public ExampleFrameListener   // 公有继承例子中帧监听
{
public:
		TutorialFrameListener(RenderWindow *win,Camera *cam,SceneManager *sceneMgr)
			:ExampleFrameListener(win,cam,false,false)
	{
		mMouseDown=false;
		mToggle=0.0;
		mCamNode=cam->getParentSceneNode();
		mSceneMgr=sceneMgr;
		//设置旋转和移动速度
		mRotate=0.13;
		mMove=250;
	}

	bool frameStarted(const FrameEvent &evt)
	{
		mMouse->capture();
		mKeyboard->capture();
		//按Escape键时程序退出
		if(mKeyboard->isKeyDown(OIS::KC_ESCAPE))
			return false;
		bool currMouse=mMouse->getMouseState().buttonDown(OIS::MB_Left);  //按键不好用可以用鼠标
		if(currMouse&&mMouseDown)
		{
		       Light *light=mSceneMgr->getLight("light1");
			   light->setVisible(!light->isVisible());
		}
		mMouseDown=currMouse;
		mToggle-=evt.timeSinceLastFrame;
		//通过1，2键来转换摄像机视口
		if((mToggle<0.0f)&&mKeyboard->isKeyDown(OIS::KC_1))
		{
			mToggle = 0.5f;
			mCamera->getParentSceneNode()->detachObject(mCamera);
			mCamNode = mSceneMgr->getSceneNode("CamNode1");
			mCamNode->attachObject(mCamera);
		}
		else if((mToggle < 0.0f)&&mKeyboard->isKeyDown(OIS::KC_2))
		{
			mToggle = 0.5f;
			mCamera->getParentSceneNode()->detachObject(mCamera);
			mCamNode=mSceneMgr->getSceneNode("CamNode2");
			mCamNode->attachObject(mCamera);
		}
		//用W,S,A,D键或上下左右键来控制相机节点的移动
		Vector3 transVector=Vector3::ZERO;
		if(mKeyboard->isKeyDown(OIS::KC_UP)||mKeyboard->isKeyDown(OIS::KC_W))
			transVector.z -= mMove;
		if(mKeyboard->isKeyDown(OIS::KC_DOWN)||mKeyboard->isKeyDown(OIS::KC_S))
			transVector.z += mMove;
		if(mKeyboard->isKeyDown(OIS::KC_A)||mKeyboard->isKeyDown(OIS::KC_LEFT))
			transVector.x -= mMove;
		if(mKeyboard->isKeyDown(OIS::KC_D)||mKeyboard->isKeyDown(OIS::KC_RIGHT))
			transVector.x += mMove;
		//可以用PageUp,PageDown键控制在y轴上的上下移动
		if(mKeyboard->isKeyDown(OIS::KC_PGDOWN))
			transVector.y -= mMove;
		if(mKeyboard->isKeyDown(OIS::KC_PGUP))
			transVector.y += mMove;
		
		//用鼠标来控制观察方向,但只当鼠标右键被按住情况时，所以先检查右键是否按下去
		/*if(mMouse->getMouseState().buttonDown(OIS::MB_Right))
		{
			mCamNode->pitch(Degree(-mRotate *mMouse->getMouseState().Y.rel),Node::TS_LOCAL);
			mCamNode->yaw(Degree(-mRotate *mMouse->getMouseState().X.rel),Node::TS_WORLD);
		
		}*/




		    return true;

		
	}

protected:
		bool mMouseDown;        
	Real mToggle;      
	Real mRotate;      
	Real mMove;    
	SceneManager *mSceneMgr;
	SceneNode *mCamNode;
};

class TutorialApplication:public ExampleApplication
{
public:
		TutorialApplication()
	{}
	~TutorialApplication()
	{}
protected:
		void createCamera(void)
	{
		mCamera = mSceneMgr->createCamera("PlayerCam");
		mCamera->setNearClipDistance(5);
		//创建帧监听
		mFrameListener = new TutorialFrameListener(mWindow,mCamera,mSceneMgr);
		mRoot->addFrameListener(mFrameListener);
		//显示帧频
		mFrameListener->showDebugOverlay(true);
	}
	void createScene(void)
	{
		//创建一个很暗的光环境
		mSceneMgr->setAmbientLight(ColourValue(0.25,0.25,0.25));
		//添加忍者
		Entity *ent=mSceneMgr->createEntity("Ninja","ninja.mesh");
		SceneNode *node=mSceneMgr->getRootSceneNode()->createChildSceneNode("NinjaNode");
		node->attachObject(ent);
		//创建一个白色光，与忍者有一小段距离
		Light *light=mSceneMgr->createLight("light1");
		light->setType(Light::LT_POINT);
		light->setPosition(Vector3(250,150,250));
		light->setDiffuseColour(ColourValue::White);      //漫反射颜色
		light->setSpecularColour(ColourValue::White);   //镜面反射的颜色
		//创建节点绑定相机
		node=mSceneMgr->getRootSceneNode()->createChildSceneNode("CamNode1",Vector3(-400,200,400));
		node->yaw(Degree(-45));
		node->attachObject(mCamera);
		//再创建一个相机
		node=mSceneMgr->getRootSceneNode()->createChildSceneNode("CamNode2",Vector3(0,200,400));
       
 
	}
	void createFrameListener(void)
	{

	}
};

#if OGRE_PLATFORM == OGRE_PLATFORM_WIN32
#define WIN32_LEAN_AND_MEAN
#include "windows.h"
INT WINAPI WinMain( HINSTANCE hInst, HINSTANCE, LPSTR strCmdLine,
				   INT )
#else
int main(int argc, char **argv)
#endif
{
	// Create application object
	TutorialApplication app;
	try {
		app.go();
	} catch( Exception& e ) {
#if OGRE_PLATFORM == OGRE_PLATFORM_WIN32
		MessageBox( NULL, e.getFullDescription().c_str(), "An exception has occured!", MB_OK | MB_ICONERROR | MB_TASKMODAL);
#else
		fprintf(stderr, "An exception has occured: %s\n",
			e.getFullDescription().c_str());
#endif
	}return 0;

}

