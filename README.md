



using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;
using Microsoft.Xna.Framework.Input.Touch;
using System;


namespace FOOTBALLCHIK
{

    public class Game1 : Game
    {
    
        private GraphicsDeviceManager _graphics;
        private SpriteBatch _spriteBatch;
        private Texture2D _backgroundTexture;
        private Texture2D _ballTexture;
        private Texture2D _goalkeeperTexture;
        private double _goalLinePosition;
        private int _screenWidth;
        private int _screenHeight;
        private Rectangle _backgroundRectangle;
        private Vector2 _ballPosition;
        private Rectangle _ballRectangle;
        private Vector2 _initialBallPosition;
        private bool _isBallMoving;
        private Vector2 _ballVelocity;
        private float _ballPositionX;
        private float _ballPositionY;
        private bool _isBallHit;
        private TimeSpan _startMovement;
        private Rectangle _goalkeeperRectangle;
        private int _goalkeeperPositionX;
        private int _goalkeeperPositionY;
        private int _goalKeeperWidth;
        private int _goalKeeperHeight;
        private double _aCoef;
        private double _deltaCoef;
        
        public Game1()
        {
            _graphics = new GraphicsDeviceManager(this);
            Content.RootDirectory = "Content";
            IsMouseVisible = true;
        }
        protected override void Initialize()
        {
            // TODO: Add your initialization logic here
            ResetWindowSize();
            Window.ClientSizeChanged += (s, e) => ResetWindowSize();
            TouchPanel.EnabledGestures = GestureType.Flick;
            base.Initialize();
        }
        protected override void LoadContent()
        {
            // Create a new SpriteBatch, which can be used to draw textures.
            _spriteBatch = new SpriteBatch(GraphicsDevice);
            // TODO: use this.Content to load your game content here
            _backgroundTexture = Content.Load<Texture2D>("SoccerField");
            _ballTexture = Content.Load<Texture2D>("SoccerBall");
            _goalkeeperTexture = Content.Load<Texture2D>("Goalkeeper");
        }
        protected override void Update(GameTime gameTime)
        {
            if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed ||
        Keyboard.GetState().IsKeyDown(Keys.Escape))
                Exit();

            // TODO: Add your update logic here
            if (!_isBallMoving && TouchPanel.IsGestureAvailable)
            {
                // Read the next gesture    
                GestureSample gesture = TouchPanel.ReadGesture();
                if (gesture.GestureType == GestureType.Flick)
                {
                    _isBallMoving = true;
                    _isBallHit = false;
                    _startMovement = gameTime.TotalGameTime;
                    _ballVelocity = gesture.Delta * (float)TargetElapsedTime.TotalSeconds / 5.0f;
                }
            }
            if (_isBallMoving)
            {
                _ballPositionX += _ballVelocity.X;
                _ballPositionY += _ballVelocity.Y;
                _goalkeeperPositionX = (int)((_screenWidth * 0.11) *
                                  Math.Sin(_aCoef * gameTime.TotalGameTime.TotalMilliseconds +
                                  _deltaCoef) + (_screenWidth * 0.75) / 2.0 + _screenWidth * 0.11);
                _ballPosition += _ballVelocity;
                // reached goal line
                var timeInMovement = (gameTime.TotalGameTime - _startMovement).TotalSeconds;

                var isTimeout = timeInMovement > 5.0;
                if (_ballPosition.Y < _goalLinePosition || isTimeout)
                {
                    bool isGoal = !isTimeout &&
                        (_ballPosition.X > _screenWidth * 0.375) &&
                        (_ballPosition.X < _screenWidth * 0.623);
                    ResetGame();
                }
                _ballRectangle.X = (int)_ballPosition.X;
                _ballRectangle.Y = (int)_ballPosition.Y;
            }
            base.Update(gameTime);

            TouchCollection touches = TouchPanel.GetState();

            if (touches.Count > 0 && touches[0].State == TouchLocationState.Pressed)
            {
                var touchPoint = new Point((int)touches[0].Position.X, (int)touches[0].Position.Y);
                var hitRectangle = new Rectangle((int)_ballPositionX, (int)_ballPositionY, _ballTexture.Width,
                    _ballTexture.Height);
                hitRectangle.Inflate(20, 20);
                _isBallHit = hitRectangle.Contains(touchPoint);
            }
            _ballRectangle.X = (int)_ballPosition.X;
            _ballRectangle.Y = (int)_ballPosition.Y;
            _goalkeeperRectangle = new Rectangle(_goalkeeperPositionX, _goalkeeperPositionY,
                             _goalKeeperWidth, _goalKeeperHeight);
            if (_goalkeeperRectangle.Intersects(_ballRectangle))
            {
                ResetGame();
            }
            base.Update(gameTime);
            if (gesture.GestureType == GestureType.Flick)
            {
                _isBallMoving = true;
                _isBallHit = false;
                _startMovement = gameTime.TotalGameTime;
                _ballVelocity = gesture.Delta * (float)TargetElapsedTime.TotalSeconds / 5.0f;
                var rnd = new Random();
                _aCoef = rnd.NextDouble() * 0.005;
                _deltaCoef = rnd.NextDouble() * Math.PI / 2;
            }
        }

        protected override void Draw(GameTime gameTime)
        {
            GraphicsDevice.Clear(Color.Green);

            // Begin a sprite batch    
            _spriteBatch.Begin();
            // Draw the background    
            _spriteBatch.Draw(_backgroundTexture, _backgroundRectangle, Color.White);
            // Draw the ball
            _spriteBatch.Draw(_ballTexture, _ballRectangle, Color.White);
            // Draw the goalkeeper
            _spriteBatch.Draw(_goalkeeperTexture, _goalkeeperRectangle, Color.White);
            // End the sprite batch    
            _spriteBatch.End();
            base.Draw(gameTime);
        }

        private Vector2 Get_initialBallPosition()
        {
            return _initialBallPosition;
        }

        private void ResetWindowSize(Vector2 _initialBallPosition)
        {
            _goalkeeperPositionX = (_screenWidth - _goalKeeperWidth) / 2;
            _goalkeeperPositionY = (int)(_screenHeight * 0.12);
            _goalKeeperWidth = (int)(_screenWidth * 0.05);
            _goalKeeperHeight = (int)(_screenWidth * 0.005);
            _goalLinePosition = _screenHeight * 0.05;
            _screenWidth = Window.ClientBounds.Width;
            _screenHeight = Window.ClientBounds.Height;
            _backgroundRectangle = new Rectangle(0, 0, _screenWidth, _screenHeight);
            _initialBallPosition = new Vector2(_screenWidth / 2.0f, _screenHeight * 0.8f);
            var ballDimension = (_screenWidth > _screenHeight) ?
                (int)(_screenWidth * 0.02) :
                (int)(_screenHeight * 0.035);
            _ballPosition = (int)_initialBallPosition.Y;
            _ballRectangle = new Rectangle((int)_initialBallPosition.X, (int)_initialBallPosition.Y,
                ballDimension, ballDimension);
        }
        private void ResetGame()
        {
            _ballPosition = new Vector2(_initialBallPosition.X, _initialBallPosition.Y);
            _goalkeeperPositionX = (_screenWidth - _goalKeeperWidth) / 2;
            _isBallMoving = false;
            _isBallHit = false;
            while (TouchPanel.IsGestureAvailable)
                TouchPanel.ReadGesture();
        }
    }

}
