FROM python:3.10
# Change to root so that we can add stuff to the container
USER root
# Update repository and add convenience programs
RUN apt-get update && apt-get upgrade -y && apt-get install ncurses-bin curl tmux vim -y
# Setup code directory
RUN mkdir -p /opt/src
WORKDIR /opt/src
# Add requirements file and install in local development environment
COPY requirements.txt /opt/src
RUN pip install -r /opt/src/requirements.txt

EXPOSE 8888

# Start the Jupyter server
CMD ["jupyter-notebook","--ip","0.0.0.0", "--no-browser", "--allow-root"]