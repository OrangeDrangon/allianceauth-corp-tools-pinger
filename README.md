# High Performance Pings

Leverage the corptools data to notify via discord certain events at a corp/alliance level

filter on/off regions/const/system/corps/alliances/types/strucutre type/notification type via admin. end specific notifications to different places via webhooks

configurable @ settings

# What Pings are Available:

## Structures

- attack/reinforce
  - StructureLostShields
  - StructureLostArmor
  - StructureUnderAttack
- low fuel ()
- abandoned ()
- destroyed (StructureDestroyed)
- low power (StructureWentLowPower)
- anchoring (StructureAnchoring)
- unanchoring (StructureUnanchoring)
- high power (StructureWentHighPower)
- transfer (OwnershipTransferred)

## POS

- attack/reinforce
  - TowerAlertMsg

## Sov

- attacks
  - SovStructureReinforced
  - EntosisCaptureStarted
- pos anchoring (AllAnchoringMsg)

## Moons

- Extraction Started (MoonminingExtractionStarted)
- Extraction Complete (MoonminingExtractionFinished)
- Laser Fired (MoonminingLaserFired)
- auto fracture (MoonminingAutomaticFracture)

## HR

- New application (CorpAppNewMsg)

# Setup

add periodic task, default timing below.

`pinger.tasks.bootstrap_notification_tasks`

make a new cron `*/10 * * * * *`

# Optimization

## Separate Worker Queue

Edit `myauth/myauth/celery.py`

```python
app.conf.task_routes = {.....
                        'pinger.tasks.corporation_notification_update': {'queue':'pingbot'},
                        .....
                        }
```

## Bare Metal
Add program block to `supervisor.conf`

```ini
[program:pingbot]
command=/path/to/venv/venv/bin/celery -A myauth worker --pool=threads --concurrency=5 -Q pingbot
directory=/home/allianceserver/myauth
user=allianceserver
numprocs=1
stdout_logfile=/home/allianceserver/myauth/log/pingbot.log
stderr_logfile=/home/allianceserver/myauth/log/pingbot.log
autostart=true
autorestart=true
startsecs=10
stopwaitsecs=60
killasgroup=true
priority=998
```
## Docker
add a new worker container

```yaml
  allianceauth_worker_pingbot:
    container_name: allianceauth_worker_pingbot
    <<: [*allianceauth-base, *allianceauth-health-checks]
    entrypoint: ["celery","-A","myauth","worker","--pool=threads","--concurrency=10","-Q","pingbot","-n","P_%n"]
```
