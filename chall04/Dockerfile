FROM ubuntu:22.04
RUN apt-get update && apt-get install -y gcc
RUN echo "secret-host-data" > /etc/host-secret-simulated   # simulate a host file readable via open_by_handle_at
CMD ["/bin/bash"]
