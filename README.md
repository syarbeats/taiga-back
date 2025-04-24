## Taiga Backend

[![Managed with Taiga.io](https://img.shields.io/badge/managed%20with-TAIGA.io-709f14.svg)](https://tree.taiga.io/project/taiga/ "Managed with Taiga.io")
[![Tests Status](https://github.com/taigaio/taiga-back/workflows/Taiga%20Back%20-%20Test%20and%20Coverage/badge.svg?branch=main)](https://github.com/taigaio/taiga-back/actions?query=workflow%3A%22Taiga+Back+-+Test+and+Coverage%22 "Tests Status")
[![Coverage Status](https://img.shields.io/coveralls/taigaio/taiga-back/main.svg)](https://coveralls.io/r/taigaio/taiga-back?branch=main "Coverage Status")

## Documentation

Currently, we have authored three main documentation hubs:

- **[API](https://docs.taiga.io/api.html)**: Our API documentation and reference for developing from Taiga API.
- **[Documentation](https://docs.taiga.io/)**: If you need to install Taiga on your own server, this is the place to find some guides.
- **[Taiga Resources](https://community.taiga.io/)**: This page is intended to be the support reference page for the users.

## Running Taiga

### Using Docker (recommended)

The easiest way to run Taiga is using Docker and Docker Compose:

```bash
docker-compose up
```

This will start all the required services (PostgreSQL, RabbitMQ, and Taiga backend).

### Without Docker

If you prefer to run Taiga without Docker, follow these steps:

#### Prerequisites

- Python 3.8+
- PostgreSQL 12
- RabbitMQ 3.8+
- pip and virtualenv

#### Setup

1. **Create and activate a virtual environment**:

   ```bash
   python -m venv taiga-env
   source taiga-env/bin/activate  # On Windows: taiga-env\Scripts\activate
   ```

2. **Install dependencies**:

   ```bash
   pip install -r requirements.txt
   ```

3. **Set up PostgreSQL**:

   Create a PostgreSQL database for Taiga:

   ```bash
   createuser taiga
   createdb taiga -O taiga
   ```

   Set a password for the taiga user:

   ```bash
   psql -c "ALTER USER taiga WITH PASSWORD 'taiga';"
   ```

4. **Configure Taiga**:

   Create a configuration file by copying the example:

   ```bash
   cp settings/config.py.dev.example settings/config.py
   ```

   Edit the `settings/config.py` file to match your environment, particularly the database settings:

   ```python
   DATABASES = {
       'default': {
           'ENGINE': 'django.db.backends.postgresql',
           'NAME': 'taiga',
           'USER': 'taiga',
           'PASSWORD': 'taiga',
           'HOST': 'localhost',
           'PORT': '5432',
       }
   }
   ```

   If you're using RabbitMQ for events and async tasks, configure the RabbitMQ settings:

   ```python
   EVENTS_PUSH_BACKEND = "taiga.events.backends.rabbitmq.EventsPushBackend"
   EVENTS_PUSH_BACKEND_OPTIONS = {
       "url": "amqp://taiga:taiga@localhost:5672/taiga"
   }

   CELERY_ENABLED = True
   CELERY_BROKER_URL = "amqp://taiga:taiga@localhost:5672/taiga"
   ```

5. **Initialize the database**:

   ```bash
   python manage.py migrate
   python manage.py loaddata initial_user
   python manage.py loaddata initial_project_templates
   python manage.py compilemessages
   python manage.py collectstatic --noinput
   ```

6. **Create a superuser (admin)**:

   ```bash
   python manage.py createsuperuser
   ```

7. **Run the development server**:

   ```bash
   python manage.py runserver
   ```

   The Taiga API will be available at http://localhost:8000/api/v1/

8. **For production environments**:

   For production use, you should:
   - Use a proper WSGI server like Gunicorn
   - Set up a reverse proxy with Nginx
   - Configure SSL
   - Set DEBUG=False in your settings

   Example with Gunicorn:
   ```bash
   pip install gunicorn
   gunicorn --workers=4 --timeout=60 --log-level=info --access-logfile=- --error-logfile=- taiga.wsgi
   ```

#### Running Celery (for async tasks)

If you've enabled Celery for asynchronous tasks, you'll need to run the Celery worker:

```bash
celery -A taiga worker -l info
```

## Bug reports

If you **find a bug** in Taiga you can always report it:

- in [Taiga issues](https://tree.taiga.io/project/taiga/issues). **This is the preferred way**
- in [Github issues](https://github.com/taigaio/taiga-back/issues)
- send us a mail to support@taiga.io if is a bug related to [tree.taiga.io](https://tree.taiga.io)
- send us a mail to security@taiga.io if is a **security bug**

One of our fellow Taiga developers will search, find and hunt it as soon as possible.

Please, before reporting a bug, write down how can we reproduce it, your operating system, your browser and version, and if it's possible, a screenshot. Sometimes it takes less time to fix a bug if the developer knows how to find it.

## Community

If you **need help to setup Taiga**, want to **talk about some cool enhancemnt** or you have **some questions**, please go to [Taiga community](https://community.taiga.io/).

If you want to be up to date about announcements of releases, important changes and so on, you can subscribe to our newsletter (you will find it by scrolling down at [https://taiga.io](https://www.taiga.io/)) and follow [@taigaio](https://twitter.com/taigaio) on Twitter.

## Contribute to Taiga

There are many different ways to contribute to Taiga's platform, from patches, to documentation and UI enhancements, just find the one that best fits with your skills. Check out our detailed [contribution guide](https://community.taiga.io/t/how-can-i-contribute/159)

## Code of Conduct

Help us keep the Taiga Community open and inclusive. Please read and follow our [Code of Conduct](https://github.com/taigaio/code-of-conduct/blob/main/CODE_OF_CONDUCT.md).

## License

Every code patch accepted in Taiga codebase is licensed under [MPL 2.0](LICENSE). You must be careful to not include any code that can not be licensed under this license.

Please read carefully [our license](LICENSE) and ask us if you have any questions as well as the [Contribution policy](https://github.com/taigaio/taiga-back/blob/main/CONTRIBUTING.md).
