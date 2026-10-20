#include <GL/freeglut.h>
#include <cmath>
#include <cstdlib>
#include <ctime>
#include <vector>
using namespace std;

//   SETTINGS

const float WORLD = 1000.0f;
const float SPEED = 0.22f;
const float JUMP = 0.32f;
const float GRAVITY = 0.015f;

float px = 0, py = 0, pz = 0;
float angle = 0;
float velocityY = 0;
bool jumping = false;
bool finished = false;

bool keys[256] = {};
bool special[256] = {};

struct Object {
    float x, z, size;
    int type;
};

vector<Object> trees, rocks, flowers;

//POND

const float pondX = 12, pondZ = 8;
const float pondRX = 10, pondRZ = 7;

const int STONES = 7;

float bridgeX[STONES] = {12,10.5,13.5,10.5,13.5,10.5,12};
float bridgeZ[STONES] = {1.5,3.5,5.5,7.5,9.5,11.5,13.5};

const float finishX = 12;
const float finishZ = 17;

//BASIC FUNCTIONS

float randomFloat(float a, float b)
{
    return a + (float)rand() / RAND_MAX * (b - a);
}

void cube(float x, float y, float z,
          float sx, float sy, float sz)
{
    glPushMatrix();
    glTranslatef(x,y,z);
    glScalef(sx,sy,sz);
    glutSolidCube(1);
    glPopMatrix();
}

//   CHECK AREAS

bool inStart(float x, float z)
{
    return fabs(x) < 10 && fabs(z) < 10;
}

bool inPond(float x, float z)
{
    float dx = x - pondX;
    float dz = z - pondZ;

    return dx*dx/(pondRX*pondRX) +
           dz*dz/(pondRZ*pondRZ) < 1;
}

bool onBridge(float x, float z)
{
    for(int i=0;i<STONES;i++)
    {
        if(fabs(x-bridgeX[i]) < 1.7 &&
           fabs(z-bridgeZ[i]) < 1.25)
            return true;
    }

    return false;
}

//   WORLD OBJECTS

void drawTree(float x,float z,float s)
{
    glPushMatrix();
    glTranslatef(x,0,z);
    glScalef(s,s,s);

    glColor3f(0.42,0.22,0.08);
    cube(0,1.8,0,0.9,3.6,0.9);

    glColor3f(0.04,0.42,0.07);
    cube(0,3.7,0,3.2,2,3.2);

    glColor3f(0.08,0.55,0.10);
    cube(0,5,0,2.3,1.8,2.3);

    glPopMatrix();
}

void drawFlower(float x,float z,int type)
{
    glPushMatrix();
    glTranslatef(x,0,z);

    glColor3f(0.05,0.4,0.05);
    cube(0,0.35,0,0.1,0.7,0.1);

    if(type==0)
        glColor3f(0.95,0.15,0.15);
    else if(type==1)
        glColor3f(1,0.8,0.05);
    else
        glColor3f(1,0.9,0.9);

    cube(0,0.75,0,0.35,0.35,0.35);

    glPopMatrix();
}

void drawRock(float x,float z,float s)
{
    glColor3f(0.35,0.35,0.35);
    cube(x,0.3*s,z,s,0.6*s,0.8*s);
}

//   GROUND

void drawGround()
{
    glColor3f(0.2,0.65,0.18);
    cube(0,-0.3,0,WORLD,0.5,WORLD);
}

//   POND

void drawPond()
{
    // Sand
    glColor3f(0.76,0.65,0.40);
    cube(pondX,-0.02,pondZ,28,0.08,22);

    // Water
    glColor3f(0.08,0.42,0.82);
    cube(pondX,0.06,pondZ,20,0.08,14);
}

//   BRIDGE

void drawBridge()
{
    for(int i=0;i<STONES;i++)
    {
        glColor3f(0.38,0.38,0.38);
        cube(bridgeX[i],0.18,bridgeZ[i],
             3.4,0.36,2.5);

        glColor3f(0.52,0.52,0.52);
        cube(bridgeX[i],0.38,bridgeZ[i],
             3.2,0.08,2.3);
    }
}

//   FINISH

void drawFinish()
{
    glColor3f(0.8,0.05,0.05);

    cube(finishX-3,2,finishZ,
         0.35,4,0.35);

    cube(finishX+3,2,finishZ,
         0.35,4,0.35);

    glColor3f(0.95,0.8,0.05);

    cube(finishX,3.7,finishZ,
         6,0.35,0.35);

    for(int i=0;i<6;i++)
    {
        if(i%2==0)
            glColor3f(1,1,1);
        else
            glColor3f(0.05,0.05,0.05);

        cube(finishX-2.5+i,
             0.08,
             finishZ,
             1,0.05,0.8);
    }
}

bool reachedFinish()
{
    float dx=px-finishX;
    float dz=pz-finishZ;

    return sqrt(dx*dx+dz*dz)<2.8;
}

//   COLLISION

bool hitTree(float x,float z)
{
    for(int i=0;i<(int)trees.size();i++)
    {
        float dx=x-trees[i].x;
        float dz=z-trees[i].z;

        if(sqrt(dx*dx+dz*dz)<trees[i].size)
            return true;
    }

    return false;
}

bool hitRock(float x,float z)
{
    for(int i=0;i<(int)rocks.size();i++)
    {
        float dx=x-rocks[i].x;
        float dz=z-rocks[i].z;

        if(sqrt(dx*dx+dz*dz)<rocks[i].size)
            return true;
    }

    return false;
}

bool blocked(float x,float z)
{
    if(x < -495 || x > 495 ||
       z < -495 || z > 495)
        return true;

    if(hitTree(x,z))
        return true;

    if(hitRock(x,z))
        return true;

    if(inPond(x,z) && !onBridge(x,z))
        return true;

    return false;
}

//   WORLD GENERATION

void generateWorld()
{
    trees.clear();
    rocks.clear();
    flowers.clear();

    srand((unsigned)time(NULL));

    // Trees
    for(int i=0;i<180;i++)
    {
        Object t;

        t.x=randomFloat(-500,500);
        t.z=randomFloat(-500,500);
        t.size=randomFloat(0.75,1.35);
        t.type=0;

        if(inStart(t.x,t.z) || inPond(t.x,t.z))
        {
            i--;
            continue;
        }

        trees.push_back(t);
    }

    // Flowers
    for(int i=0;i<250;i++)
    {
        Object f;

        f.x=randomFloat(-500,500);
        f.z=randomFloat(-500,500);
        f.size=1;
        f.type=rand()%3;

        if(inStart(f.x,f.z) || inPond(f.x,f.z))
        {
            i--;
            continue;
        }

        flowers.push_back(f);
    }

    // Rocks
    for(int i=0;i<100;i++)
    {
        Object r;

        r.x=randomFloat(-500,500);
        r.z=randomFloat(-500,500);
        r.size=randomFloat(0.5,1.5);
        r.type=0;

        if(inStart(r.x,r.z) || inPond(r.x,r.z))
        {
            i--;
            continue;
        }

        rocks.push_back(r);
    }
}

//   PLAYER

void drawPlayer()
{
    glPushMatrix();

    glTranslatef(px,py,pz);
    glRotatef(angle,0,1,0);

    // Body
    glColor3f(0.1,0.35,0.85);
    cube(0,1.8,0,1.2,1.8,0.65);

    // Head
    glColor3f(0.85,0.62,0.42);
    cube(0,3.25,0,1.1,1.1,1.1);

    // Hair
    glColor3f(0.12,0.06,0.02);
    cube(0,3.78,0,1.15,0.25,1.15);

    // Arms
    glColor3f(0.85,0.62,0.42);

    cube(-0.85,1.8,0,0.4,1.6,0.5);
    cube(0.85,1.8,0,0.4,1.6,0.5);

    // Legs
    glColor3f(0.08,0.12,0.35);

    cube(-0.32,0.55,0,0.5,1.1,0.6);
    cube(0.32,0.55,0,0.5,1.1,0.6);

    glPopMatrix();
}

//   MOVEMENT

void updatePlayer()
{
    float mx=0,mz=0;

    if(keys['w'] || keys['W'] || special[GLUT_KEY_UP])
        mz-=1;

    if(keys['s'] || keys['S'] || special[GLUT_KEY_DOWN])
        mz+=1;

    if(keys['a'] || keys['A'] || special[GLUT_KEY_LEFT])
        mx-=1;

    if(keys['d'] || keys['D'] || special[GLUT_KEY_RIGHT])
        mx+=1;

    float len=sqrt(mx*mx+mz*mz);

    if(len>0)
    {
        mx/=len;
        mz/=len;

        if(mx>0.1) angle=90;
        else if(mx<-0.1) angle=-90;
        else if(mz<0) angle=0;
        else angle=180;
    }

    float nx=px+mx*SPEED;
    float nz=pz+mz*SPEED;

    if(!blocked(nx,nz))
    {
        px=nx;
        pz=nz;
    }

    if(!finished && reachedFinish())
        finished=true;

    // Jump
    if(jumping)
    {
        py+=velocityY;
        velocityY-=GRAVITY;

        if(py<=0)
        {
            py=0;
            velocityY=0;
            jumping=false;
        }
    }
}

//   SKY

float cloudX=-80;

void drawCloud(float x,float y,float z)
{
    glColor3f(1,1,1);

    cube(x,y,z,5,1.5,2);
    cube(x-2,y+0.5,z,3,1.5,2);
    cube(x+2,y+0.3,z,3,1.5,2);
}

void drawSky()
{
    glDisable(GL_LIGHTING);

    // Sun
    glColor3f(1,0.85,0.1);

    glPushMatrix();
    glTranslatef(-35,35,-50);
    glutSolidSphere(5,20,20);
    glPopMatrix();

    drawCloud(cloudX,25,-30);
    drawCloud(cloudX+40,28,-70);
    drawCloud(cloudX-50,23,20);

    glEnable(GL_LIGHTING);
}

void updateClouds()
{
    cloudX+=0.02;

    if(cloudX>150)
        cloudX=-150;
}

//   TEXT

void text(float x,float y,const char* s)
{
    glRasterPos2f(x,y);

    for(int i=0;s[i];i++)
        glutBitmapCharacter(
            GLUT_BITMAP_HELVETICA_18,
            s[i]
        );
}

void message()
{
    glMatrixMode(GL_PROJECTION);
    glPushMatrix();
    glLoadIdentity();

    gluOrtho2D(0,1200,0,700);

    glMatrixMode(GL_MODELVIEW);
    glPushMatrix();
    glLoadIdentity();

    glDisable(GL_LIGHTING);

    if(finished)
    {
        glColor3f(1,0.85,0.05);
        text(470,620,
             "FINISH! You crossed the bridge!");
    }
    else
    {
        glColor3f(1,1,1);
        text(20,660,
             "WASD / Arrow Keys = Move    SPACE = Jump");
    }

    glEnable(GL_LIGHTING);

    glPopMatrix();

    glMatrixMode(GL_PROJECTION);
    glPopMatrix();

    glMatrixMode(GL_MODELVIEW);
}

//   CAMERA

void camera()
{
    glMatrixMode(GL_MODELVIEW);
    glLoadIdentity();

    gluLookAt(
        px,py+5,pz+8,
        px,py+1.7,pz,
        0,1,0
    );
}

//   LIGHT

void lighting()
{
    GLfloat pos[]={-30,40,-40,1};
    GLfloat ambient[]={0.45,0.45,0.45,1};
    GLfloat diffuse[]={1,1,1,1};

    glEnable(GL_LIGHTING);
    glEnable(GL_LIGHT0);

    glLightfv(GL_LIGHT0,GL_POSITION,pos);
    glLightfv(GL_LIGHT0,GL_AMBIENT,ambient);
    glLightfv(GL_LIGHT0,GL_DIFFUSE,diffuse);

    glEnable(GL_COLOR_MATERIAL);

    glColorMaterial(
        GL_FRONT,
        GL_AMBIENT_AND_DIFFUSE
    );
}

//   WORLD

void drawWorld()
{
    drawGround();
    drawPond();
    drawBridge();
    drawFinish();

    for(int i=0;i<(int)trees.size();i++)
        drawTree(
            trees[i].x,
            trees[i].z,
            trees[i].size
        );

    for(int i=0;i<(int)flowers.size();i++)
        drawFlower(
            flowers[i].x,
            flowers[i].z,
            flowers[i].type
        );

    for(int i=0;i<(int)rocks.size();i++)
        drawRock(
            rocks[i].x,
            rocks[i].z,
            rocks[i].size
        );
}

//   DISPLAY

void display()
{
    glClear(
        GL_COLOR_BUFFER_BIT |
        GL_DEPTH_BUFFER_BIT
    );

    camera();
    lighting();

    drawWorld();
    drawPlayer();
    drawSky();
    message();

    glutSwapBuffers();
}

//   RESIZE

void reshape(int w,int h)
{
    if(h==0) h=1;

    glViewport(0,0,w,h);

    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();

    gluPerspective(
        60,
        (double)w/h,
        0.1,
        1000
    );

    glMatrixMode(GL_MODELVIEW);
}

//   KEYBOARD

void keyDown(unsigned char key,int x,int y)
{
    keys[key]=true;

    if(key==' ')
    {
        if(!jumping)
        {
            jumping=true;
            velocityY=JUMP;
        }
    }

    if(key==27)
        exit(0);
}

void keyUp(unsigned char key,int x,int y)
{
    keys[key]=false;
}

void specialDown(int key,int x,int y)
{
    if(key<256)
        special[key]=true;
}

void specialUp(int key,int x,int y)
{
    if(key<256)
        special[key]=false;
}

//   TIMER

void timer(int value)
{
    updatePlayer();
    updateClouds();

    glutPostRedisplay();
    glutTimerFunc(16,timer,0);
}

//   OPENGL

void init()
{
    glClearColor(
        0.45,0.75,1,1
    );

    glEnable(GL_DEPTH_TEST);
    glEnable(GL_NORMALIZE);

    glShadeModel(GL_SMOOTH);

    lighting();
}

//   MAIN

int main(int argc,char** argv)
{
    glutInit(&argc,argv);

    glutInitDisplayMode(
        GLUT_DOUBLE |
        GLUT_RGB |
        GLUT_DEPTH
    );

    glutInitWindowSize(1200,700);

    glutCreateWindow(
        "Open World Garden Adventure"
    );

    init();
    generateWorld();

    glutDisplayFunc(display);
    glutReshapeFunc(reshape);

    glutKeyboardFunc(keyDown);
    glutKeyboardUpFunc(keyUp);

    glutSpecialFunc(specialDown);
    glutSpecialUpFunc(specialUp);

    glutTimerFunc(16,timer,0);

    glutMainLoop();

    return 0;
}
