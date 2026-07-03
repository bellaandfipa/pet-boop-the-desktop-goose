# pet-boop-the-desktop-goose
now ya can pet/boop the goose
left click for boop
long left click for pet
now ya can pet
or boop
your desktop goose
hjonk hjonk

csharp

using System;
using GooseShared;

namespace DesktopGoosePetMod
{
    public class ModMain : IMod
    {
        private Random random = new Random();
        
        // Tracks the exact time when the left click started
        private DateTime clickStartTime;
        private bool isHoldingLeftClick = false;

        public void Init()
        {
            // We hook into the game's frame update to watch the mouse continuously
            InjectionPoints.PostTick += OnTick;
        }

        private void OnTick(GooseWall goose)
        {
            // 1. Check if the mouse cursor is actually hovering over the goose
            if (Input.IsMouseOverGoose(goose))
            {
                // 2. Detect the exact moment the Left Mouse Button is first pressed down
                if (Input.MouseButtonPressed(Input.MouseButtons.Left) && !isHoldingLeftClick)
                {
                    isHoldingLeftClick = true;
                    clickStartTime = DateTime.Now;
                }

                // 3. Detect the moment the Left Mouse Button is released
                if (!Input.MouseButtonPressed(Input.MouseButtons.Left) && isHoldingLeftClick)
                {
                    isHoldingLeftClick = false;
                    TimeSpan holdDuration = DateTime.Now - clickStartTime;

                    // If held for longer than 500 milliseconds (half a second), it's a Pet. Otherwise, a Boop.
                    if (holdDuration.TotalMilliseconds >= 500)
                    {
                        TriggerPet(goose);
                    }
                    else
                    {
                        TriggerBoop(goose);
                    }
                }
            }
            else
            {
                // If the player moves the mouse off the goose while holding, reset the tracker
                if (!Input.MouseButtonPressed(Input.MouseButtons.Left))
                {
                    isHoldingLeftClick = false;
                }
            }
        }

        private void TriggerBoop(GooseWall goose)
        {
            // 30% chance to get bitten on a quick boop
            if (random.Next(0, 100) < 30)
            {
                GooseBite(goose);
            }
            else
            {
                // Startle the goose!
                goose.SetState(GooseWall.State.Angry); 
            }
        }

        private void TriggerPet(GooseWall goose)
        {
            // Lower bite chance for petting (10%), but it can still happen!
            if (random.Next(0, 100) < 10)
            {
                GooseBite(goose);
            }
            else
            {
                // The goose enjoys it and wanders off peacefully
                goose.SetState(GooseWall.State.Wandering);
            }
        }

        private void GooseBite(GooseWall goose)
        {
            // This forces the goose to chase down and grab your cursor
            goose.SetState(GooseWall.State.NabbingMouse);
        }
    }
}
