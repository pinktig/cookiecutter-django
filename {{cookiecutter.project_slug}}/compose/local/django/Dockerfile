FROM python:3.7.6-alpine3.11

ENV PYTHONUNBUFFERED 1

RUN apk update \
  # psycopg2 dependencies
  && apk add --virtual build-deps gcc python3-dev musl-dev \
  # 쥬피터 노트북 dependencies
  && apk add build-base libzmq zeromq-dev libxslt-dev libxml2 \
  && apk add postgresql-dev \
  # Pillow dependencies
  && apk add jpeg-dev zlib-dev freetype-dev lcms2-dev openjpeg-dev tiff-dev tk-dev tcl-dev \
  # CFFI dependencies
  && apk add libffi-dev py-cffi \
  # Translations dependencies
  && apk add gettext \
  # https://docs.djangoproject.com/en/dev/ref/django-admin/#dbshell
  && apk add postgresql-client \
  # https://github.com/Belval/pdf2image#linux
  && apk add poppler-utils \
  # https://weasyprint.readthedocs.io/en/stable/install.html#alpine
  && apk add cairo-dev pango-dev gdk-pixbuf-dev

{% if cookiecutter.timezone != "UTC" %}
# 타임존 설정
RUN apk --no-cache add tzdata && \
  cp /usr/share/zoneinfo/{{ cookiecutter.timezone }} /etc/localtime \
  && echo "{{ cookiecutter.timezone }}" > /etc/timezone

{% endif %}
# Requirements are installed here to ensure they will be cached.
COPY ./requirements /requirements
RUN pip install --upgrade pip
RUN pip install -r /requirements/local.txt

COPY ./compose/production/django/entrypoint /entrypoint
RUN sed -i 's/\r$//g' /entrypoint
RUN chmod +x /entrypoint

RUN mkdir /root/.ptpython
COPY ./compose/production/django/.ptpython/config.py /root/.ptpython

RUN mkdir /root/.ptpython
COPY ./compose/production/django/.ptpython/config.py /root/.ptpython

RUN jupyter contrib nbextension install --user
RUN jupyter nbextensions_configurator enable --user

# https://github.com/lambdalisue/jupyter-vim-binding/wiki/Installation#add-vim_binding-to-ipython-notebook-extensions
RUN mkdir -p $(jupyter --data-dir)/nbextensions
RUN git clone https://github.com/lambdalisue/jupyter-vim-binding $(jupyter --data-dir)/nbextensions/vim_binding
RUN chmod -R go-w $(jupyter --data-dir)/nbextensions/vim_binding
RUN jupyter nbextension enable vim_binding/vim_binding

# https://github.com/dunovank/jupyter-themes#command-line-usage
RUN jt -t chesterish -f iosevka -fs 13 -nfs 15 -ofs 13 -vim -cellw 90%


COPY ./compose/local/django/start /start
RUN sed -i 's/\r$//g' /start
RUN chmod +x /start
{% if cookiecutter.use_celery == "y" %}
COPY ./compose/local/django/celery/worker/start /start-celeryworker
RUN sed -i 's/\r$//g' /start-celeryworker
RUN chmod +x /start-celeryworker

COPY ./compose/local/django/celery/beat/start /start-celerybeat
RUN sed -i 's/\r$//g' /start-celerybeat
RUN chmod +x /start-celerybeat

COPY ./compose/local/django/celery/flower/start /start-flower
RUN sed -i 's/\r$//g' /start-flower
RUN chmod +x /start-flower
{% endif %}
WORKDIR /app

ENTRYPOINT ["/entrypoint"]
