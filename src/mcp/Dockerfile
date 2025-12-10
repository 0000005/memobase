FROM 2brc2u4m1iawic.xuanyuan.run/python:3.12-slim

ARG PORT=8050

WORKDIR /app

# Install uv
RUN pip install uv -i https://pypi.tuna.tsinghua.edu.cn/simple

# Copy the MCP server files
COPY . .

# Install packages
RUN python -m venv .venv
RUN uv pip install -e . -i https://pypi.tuna.tsinghua.edu.cn/simple

EXPOSE ${PORT}

# Command to run the MCP server
CMD ["uv", "run", "src/main.py"]